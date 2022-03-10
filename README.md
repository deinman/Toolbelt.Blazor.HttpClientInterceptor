# Blazor WebAssembly (client-side) HttpClient Interceptor [![NuGet Package](https://img.shields.io/nuget/v/Toolbelt.Blazor.HttpClientInterceptor.svg)](https://www.nuget.org/packages/Toolbelt.Blazor.HttpClientInterceptor/)

## Summary

The class library that intercept all of the sending HTTP requests on a client side Blazor WebAssembly application.

## Supported Blazor versions

"Blazor WebAssembly App (client-side) HttpClient Interceptor" ver.9.x supports Blazor WebAssembly App version 3.2 Preview 2+, and Release Candidates, **of course, 3.2.x official release,** and **.NET 5, 6, 7 are also supported.**

## How to install and use?

**Step.1** Install the library via NuGet package, like this.

```shell
> dotnet add package Toolbelt.Blazor.HttpClientInterceptor
```

**Step.2** Register `IHttpClientInterceptor` service into the DI container, at `Main()` method in the `Program` class of your Blazor application.

```csharp
using Toolbelt.Blazor.Extensions.DependencyInjection; // <- Add this, and...
...
public class Program
{
  public static async Task Main(string[] args)
  {
    var builder = WebAssemblyHostBuilder.CreateDefault(args);
    builder.RootComponents.Add<App>("app");
    builder.Services.AddHttpClientInterceptor(); // <- Add this!
    ...
```

**Step.3** Add invoking `EnableIntercept(IServiceProvider)` extension method at the registration of `HttpClient` to DI container.

```csharp
public class Program
{
  public static async Task Main(string[] args)
  {
    ...
    builder.Services.AddScoped(sp => new HttpClient { 
      BaseAddress = new Uri(builder.HostEnvironment.BaseAddress) 
    }.EnableIntercept(sp)); // <- Add this!
    ...
```

That's all.

You can subscribe the events that will occur before/after sending all of the HTTP requests, at anywhere you can get `IHttpClientInterceptor` service from the DI container.

```csharp
@using Toolbelt.Blazor
@inject IHttpClientInterceptor Interceptor
...
@code {
  protected override void OnInitialized()
  {
    this.Interceptor.BeforeSend += Interceptor_BeforeSend;
    ...
  }

  void Interceptor_BeforeSend(object sender, HttpClientInterceptorEventArgs e)
  {
    // Do something here what you want to do.
  }
  ...
```

If you want to do async operations inside the event handler, please subscribe async version events such as `BeforeSendAsync` and `AfterSendAsync`, instead of `BeforeSend` and `AfterSend`.

> _Note:_ Please remember that if you use `IHttpClientInterceptor` to subscribe `BeforeSend`/`BeforeSendAsync`/`AfterSend`/`AfterSendAsync` events **in Blazor components (.razor),** you should unsubscribe events when the components is discarded. To do it, you should implement `IDisposable` interface in that component, like this code:

```csharp
@implements IDisposable
...
public void Dispose()
{
  this.Interceptor.BeforeSend -= Interceptor_BeforeSend;
}
```

### The arguments of event handler

The event handler for `BeforeSend`/`BeforeSendAsync`/`AfterSend`/`AfterSendAsync` events can be received `HttpClientInterceptorEventArgs` object.

The `HttpClientInterceptorEventArgs` object provides you to a request object and a response object that is come from an intercepted HttpClinet request.

```csharp
void OnAfterSend(object sender, HttpClientInterceptorEventArgs args)
{
  if (!args.Response?.IsSuccessStatusCode) {
    // Do somthing here for handle all errors of HttpClient requests.
  }
}
```

### To read the content at "AfterSendAsync" event handler

If you want to read the content in the `Response` object at the event handler for `AfterSendAsync` event, **don't** reference the `Response.Content` property **directly** to do it.

Instead, please **use the return value of the `GetCapturedContentAsync()` method.**

> _Note:_ Please remember that the `GetCapturedContentAsync()` method has a little bit performance penalty.  
> Because in the `GetCapturedContentAsync()` method, the `LoadIntoBufferAsync()` method of the `Response.Content` property is invoked.

```csharp
async Task OnAfterSendAsync(object sender, HttpClientInterceptorEventArgs args)
{
  // 👇 Don't reference "args.Response.Content" directly to read the content.
  // var content = await args.Response.Content.ReadAsStringAsync()

  // 👇 Instead, please use the return value of the "GetCapturedContentAsync()" method.
  var capturedContent = await arg.GetCapturedContentAsync();
  var content = await capturedContent.ReadAsStringAsync();
  ...
```

### Cancel a request before sending

If you want to cancel the request before sending it, you can do it by setting `false` to the `Cancel` property of the `BeforeSend`/`BeforeSendAsync` event argument.

```csharp
void OnBeforeSend(object sender, HttpClientInterceptorEventArgs args)
{
  if (/*something wron*/) {
    // 👇 Setting the "Cancel" to true will cancel sending a request.
    args.Cancel = true;
  }
}
```

In that case, a HttpClient object will return the HttpResponseMessage object with HTTP status code **"NoContent" (HTTP 204)** to the caller, and also the `AfterSend`/`AfterSendAsync` event will not be fired.

## Release Notes

Release notes is [here.](https://github.com/jsakamoto/Toolbelt.Blazor.HttpClientInterceptor/blob/master/RELEASE-NOTES.txt)

## License

[Mozilla Public License Version 2.0](https://github.com/jsakamoto/Toolbelt.Blazor.HttpClientInterceptor/blob/master/LICENSE)
