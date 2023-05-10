---
title: WebView plugin for Unity API Reference (Preview)
description: API Reference for Microsoft Mixed Reality WebView plugin for Unity.
author: michaelfarnsworth
ms.author: mifarnsworth
ms.date: 5/9/2023
ms.topic: article
keywords: HoloLens, WebView2, Unity
---
# Mixed Reality WebView Plugin for Unity API Reference (Preview)
The **Microsoft Mixed Reality WebView plugin for Unity** enables the integration of WebView2 functionality into your HoloLens 2 app.  This WebView plugin for Unity simplifies the integration of WebView2 functionality into your HoloLens 2 app by wrapping the WebView2 control, automatically handling rendering, and automatically directing input to the WebView2 control.

This plugin also manages interop between Unity and WebView2, enabling communication between JavaScript and Unity via messages and events.

This plugin exposes a subset of the functionality available via [CoreWebView2](/dotnet/api/microsoft.web.webview2.core.corewebview2).

WebView2 on HoloLens 2 and the WebView plugin for Unity are both in Preview and are subject to change before general availability.  The WebView2 Preview is available in the Insider Preview for Microsoft HoloLens.  To access this preview, you must be enrolled in the Windows Insider Program; see [Start receiving Insider builds](/hololens/hololens-insider#start-receiving-insider-builds) in _Insider preview for Microsoft HoloLens_.

WebView2 and the WebView plugin are only supported on HoloLens 2 devices running the Windows 11 update. For more information, see [Update HoloLens 2](/hololens/hololens-update-hololens).

For information on getting started with the plugin, see [Get started with WebView2 in HoloLens 2 Unity apps](/microsoft-edge/webview2/get-started/hololens2.md).

### IWebView Interface
The `IWebView` interface provide the primary interface for the plugin.

```c#
public interface IWebView
{
    event WebView_OnNavigated Navigated;

    event WebView_OnCloseRequested WindowCloseRequested;

    GameObject GameObject { get; }

    Texture2D Texture { get; }

    int Width { get; set; }

    int Height { get; set; }

    Uri Page { get; }
    
    Task OnceCreated { get; }

    void Resize(int width, int height);

    void Load(Uri url);

    void Dispose();
}
```

### IWebView Delegates
```c#
public delegate void WebView_OnNavigated(string path);

public delegate void WebView_OnCanGoForwardUpdated(bool value);
```

#### IWebView Events
##### IWebView.Navigated Event
Event triggered when [CoreWebView2.SourceChanged Event](/dotnet/api/microsoft.web.webview2.core.corewebview2.SourceChanged) is raised by WebView.

###### Example
```c#
webView.Navigated += OnNavigated;

private void OnNavigated(string uri)
{
    UrlField.text = uri;
}
```

##### IWebView.WindowCloseRequested Event
Event triggered when [CoreWebView2.WindowCloseRequested Event](/dotnet/api/microsoft.web.webview2.core.corewebview2.WindowCloseRequested) is raised by WebView.

###### Example
```c#
webView.WindowCloseRequested += OnWindowCloseRequested;

private void OnWindowCloseRequested()
{
    Destroy(GameObject);
}
```

#### IWebView Properties

##### IWebView.GameObject Property
The top-level Unity GameObject entity that represents the WebView plugin in the scene. **Readonly**.


##### IWebView.Texture Property
The 2D Unity Texture2D object that the WebView content is rendered to. **Readonly**

##### IWebView.Width Property
The width of the WebView texture and the WebView control. Note that the rendered dimensions of the IWebView instance in the Unity scene is controlled by the GameObject.

##### IWebView.Height Property
The height of the WebView texture and the WebView control. Note that the rendered dimensions of the IWebView instance in the Unity scene is controlled by the GameObject.

##### IWebView.Page Property
The URI currently loaded or being navigated to by the WebView control. **Readonly**

##### IWebView.OnceCreated Property
Task executes when the WebView control has been fully instantiated and ready to use. **Readonly**

###### Example
```c#
webView.OnceCreated.ContinueWith((task) => {

    // Finish seting up plugin.
    webview.Navigated += OnNavigated;
    webview.WindowCloseRequested += OnWindowCloseRequested;

    Load(initialURL);

}, TaskScheduler.FromCurrentSynchronizationContext());
```

#### IWebView Methods

##### IWebView.Resize Method

Changes the size of the WebView2 control and the `Texture`. For additional details, see the underlying [CoreWebView2Controller.Bounds Property](/dotnet/api/microsoft.web.webview2.core.corewebview2controller.bounds).

Note that the rendered dimensions of the IWebView instance in the Unity scene is controlled by the GameObject.

###### Example
```c#
Resize(600, 400);
```

##### IWebView.Load Method

Navigates to the specificied URI. For additional details, see the underlying [CoreWebView2.Navigate Method](/dotnet/api/microsoft.web.webview2.core.corewebview2.Navigate).

###### Example

```c#
Load(new Uri("https://www.microsoft.com"));

```

##### IWebView.Dispose Method

Clears memory, handle, callbacks, etc. related to an IWebView instance.

Invoke this method when you are done with a particular IWebView instance to ensure internal memory is properly freed. Once this is called, the IWebView instance should be considered invalid.

###### Example

```c#
void OnDestroy()
{
    webView.Dispose();
}

```

### IWithMouseEvents Interface
Interface for enable mouse/pointer input for the plugin.

```c#
public interface IWithMouseEvents
{
    void MouseEvent(WebViewMouseEventData mouseEvent);
}
```

#### IWithMouseEvents Methods

##### IWithMouseEvents.MouseEvent Method

Propagates a [WebViewMouseEventData](webview2-plugin-mouse-event-data-class.md) event to the WebView control. Depending internal logic, the event results in the [CoreWebView2Controller.SendMouseInput Method](/dotnet/api/microsoft.web.webview2.core.corewebview2compositioncontroller.sendmouseinput) or the [CoreWebView2Controller.SendPointerInput Method](/dotnet/api/microsoft.web.webview2.core.corewebview2compositioncontroller.sendpointerinput).

###### Example
```c#
public void OnPointerDown(PointerEventData eventData)
{
    IWithMouseEvents mouseEventsWebView = webView as IWithMouseEvents;

    // Call hypothetical function which converts the event's x, y into the WebView2's coordinate space.
    var hitCoord = ConvertToWebViewSpace(eventData.position.x, eventData.position.y);

    WebViewMouseEventData mouseEvent = new WebViewMouseEventData
    {
        X = hitCoord.x,
        Y = hitCoord.y,
        Type = PointerEvent.PointerDown,
        Button = PointerButton.Left,
        TertiaryAxisDeviceType = WebViewMouseEventData.TertiaryAxisDevice.PointingDevice
    };

    mouseEventsWebView.MouseEvent(mouseEvent);
}
```

### IWithPostMessage Interface
Interface for interop communication between Unity code and hosted WebView code.

To learn more about interop in WebView2 see [Interop of native-side and web-side code](/microsoft-edge/webview2/how-to/communicate-btwn-web-native).

```c#
public interface IWithPostMessage : IWebView
{
    event WebView_OnPostMessage MessageReceived;

    void PostMessage(string message, bool isJSON = false);
}
```

### IWithPostMessage Delegates

```c#
public delegate void WebView_OnPostMessage(string message);
```

#### IWithPostMessage Events

##### IWithPostMessage.MessageReceived Method
Event triggered when a new JavaScript message is received from the WebView control. For additional details, see the underlying [CoreWebView2.WebMessageReceived Event](/dotnet/api/microsoft.web.webview2.core.corewebview2.webmessagereceived).

###### Example
```c#
(webView as IWithPostMessage).MessageReceived += OnMessageReceived;

void OnMessageReceived(string message)
{
    Debug.Log(message);
}
```

#### IWithPostMessage Methods

##### IWithPostMessage.PostMessage Method

Sends a JavaScript message to the hosted content in the WebView control. Depending on the `isJSON` parameter, this will either result in a [CoreWebView2.PostWebMessageAsString Method](/dotnet/api/microsoft.web.webview2.core.corewebview2.postwebmessageasstring) or [CoreWebView2.PostWebMessageAsJson Method](/dotnet/api/microsoft.web.webview2.core.corewebview2.postwebmessageasjson).

```c#
var msg = new MyMessage("updateText", "Updated from Unity!");

(webView as IWithPostMessage).PostMessage(JsonUtility.ToJson(msg), true);
```

### IWithBrowserHistory
Interface for handling browser-history related functionality. For example, navigating to a previous page.

```c#
public interface IWithBrowserHistory : IWebView
{
    event WebView_OnCanGoForwardUpdated CanGoForwardUpdated;

    event WebView_OnCanGoBackUpdated CanGoBackUpdated;

    void GoBack();

    void GoForward();
}
```

### IWithBrowserHistory Delegates

```c#
public delegate void WebView_OnCanGoBackUpdated(bool value);

public delegate void WebView_OnCloseRequested();
```

#### IWithBrowserHistory Events

##### IWithBrowserHistory.CanGoForwardUpdated Event
Event triggered when a navigation occurs. The event delegate with provide a true value if [CoreWebView2.CanGoForward Property](/dotnet/api/microsoft.web.webview2.core.corewebview2.cangoforward) is true.

###### Example
```c#
(webView as IWithBrowserHistory).CanGoBackUpdated += OnCanGoBack;

void OnCanGoBack(bool value)
{
    BackButton.enabled = value;
}
```

##### IWithBrowserHistory.CanGoBackUpdated Event
Event triggered when a navigation occurs. The event delegate with provide a true value if [CoreWebView2.CanGoBack Property](/dotnet/api/microsoft.web.webview2.core.corewebview2.cangoback) is true.

###### Example
```c#
(webView as IWithBrowserHistory).CanGoForwardUpdated += OnCanGoForward;

void OnCanGoForward(bool value)
{
    ForwardButton.enabled = value;
}
```

#### IWithBrowserHistory Methods

##### IWithBrowserHistory.GoBack Method
Navigates to the previous page. For additional details, see the underlying [CoreWebView2.GoBack Method](/dotnet/api/microsoft.web.webview2.core.corewebview2.goback).

###### Example
```c#
(webView as IWithBrowserHistory).GoBack();
```

##### IWithBrowserHistory.GoForward Method
Navigates to the next page. For additional details, see the underlying [CoreWebView2.GoForward Method](/dotnet/api/microsoft.web.webview2.core.corewebview2.goforward).

###### Example
```c#
(webView as IWithBrowserHistory).GoForward();
```

