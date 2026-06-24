# Message Handlers

The `FhirClient` provides an option to add `HttpMessageHandler`s that hook
into the HTTP request/response cycle. These handlers let you run custom
logic immediately before a request is sent or directly after a response
is received.

## Adding extra headers

Often you need to add headers to outgoing requests (for example an
authorization token), or to execute logic on every request. Implement a
custom message handler to perform these tasks. The example below shows an
`AuthorizationMessageHandler` implementation:

```csharp
public class AuthorizationMessageHandler : HttpClientHandler
{
        public System.Net.Http.Headers.AuthenticationHeaderValue Authorization { get; set; }
        protected async override Task<HttpResponseMessage> SendAsync(HttpRequestMessage request, CancellationToken cancellationToken)
        {
                if (Authorization != null)
                        request.Headers.Authorization = Authorization;
                return await base.SendAsync(request, cancellationToken);
        }
}
```

Add that handler to a `FhirClient` instance like this:

```csharp
var handler = new AuthorizationMessageHandler();
var bearerToken = "AbCdEf123456" //example-token;
handler.Authorization = new AuthenticationHeaderValue("Bearer", bearerToken);
var client = new FhirClient(handler);
client.ReadAsync<Patient>("example");
```

## Chaining Multiple MessageHandlers

You can chain multiple `HttpMessageHandler`s to combine functionality. Use
`DelegatingHandler` to form a pipeline: each handler can inspect or
modify the request before passing it to the next handler and inspect or
modify the response returned by the inner handler. Typically the final
handler is `HttpClientHandler`, which performs the network I/O.

For example, besides an `AuthorizationMessageHandler` you might add a
`LoggingHandler`:

```csharp
public class LoggingHandler : DelegatingHandler
{
        private readonly ILogger _logger;

        public LoggingHandler(ILogger logger)
        {
                _logger = logger;
        }

        protected override async Task<HttpResponseMessage> SendAsync(HttpRequestMessage request, CancellationToken cancellationToken)
        {
                _logger.Trace("Request: " + request);
                try
                {
                        // base.SendAsync calls the inner handler
                        var response = await base.SendAsync(request, cancellationToken);
                        _logger.Trace("Response: " + response);
                        return response;
                }
                catch (Exception ex)
                {
                        _logger.Error("Failed to get response: " + ex);
                        throw;
                }
        }
}
```

Source: <https://thomaslevesque.com/2016/12/08/fun-with-the-httpclient-pipeline/>

To combine the handlers, set the `AuthorizationMessageHandler` as the
`InnerHandler` of the `LoggingHandler` (since `LoggingHandler` is a
`DelegatingHandler`):

```csharp
var authHandler = new AuthorizationMessageHandler();
var loggingHandler = new LoggingHandler()
{
        InnerHandler = authHandler
};
var client = new FhirClient("http://server.fire.ly", FhirClientSettings.CreateDefault(), loggingHandler);
```

This results in a pipeline where requests and responses flow through the
`LoggingHandler` and then the `AuthorizationMessageHandler` before
reaching the network layer.

## OnBeforeRequest and OnAfterResponse

To make use `OnBeforeRequest` and `OnAfterResponse` features that were in use on the pre-SDK5 FhirClient, you can use the pre-defined `HttpClientEventHandler`.
Use the `OnBeforeRequest` to add extra code before a request is executed by the FhirClient, and the `OnAfterResponse` event to add extra code that needs to be executed every time a response is received by the FhirClient:

```csharp
using (var handler = new HttpClientEventHandler())
{
        using (FhirClient client = new FhirClient(testEndpoint, messageHandler: handler))
        {
                handler.OnBeforeRequest += (sender, e) =>
                {
                        e.RawRequest.Headers.Authorization = new AuthenticationHeaderValue("Bearer", "Your Oauth token");
                };

                handler.OnAfterResponse += (sender, e) =>
                {
                        Console.WriteLine("Received response with status: " + e.RawResponse.StatusCode);
                };
        }
}
```
