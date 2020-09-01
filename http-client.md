# HTTP client

- [Making requests](#making-requests)
    - [Request data](#request-data)
    - [Headers](#headers)
    - [Authentication](#authentication)
    - [Error handling](#error-handling)
    - [The underlying .NET instance](#the-underlying-net-instance)
- [Testing](#testing)

Unisave provides a thin wrapper around the .NET `HttpClient` that lets you easily make HTTP requests to external web services.


<a name="making-requests"></a>
## Making requests

To make requests, you may use the `Get`, `Post`, `Put`, `Patch` and `Delete` methods. An example `GET` request is as easy as:

```cs
using Unisave.Facades;

var response = Http.Get("http://test.com");
```

You can then inspect the response:

```cs
// content
response.Body(); // string
response.Json(); // JsonObject
response.Form(); // Dictionary<string, string>
response.Bytes(); // byte[]

// status
response.Status; // int
response.IsOk; // true when status is 200
response.IsSuccessful; // true when status is 2xx
response.Failed; // true when status is 4xx or 5xx
response.IsClientError; // true when status is 4xx
response.IsServerError; // true when status is 5xx

// headers
response.Header("Content-Type"); // string
```

If the response is `application/json` or `application/x-www-form-urlencoded`, an indexer access can be used:

```cs
// get title of the latest news item for Team Fortress 2
return Http.Get(
    "https://api.steampowered.com/" +
    "ISteamNews/GetNewsForApp/v2/" +
    "?appid=440&count=1"
)["appnews"]["newsitems"][0]["title"];
```


<a name="request-data"></a>
### Request data


#### POST request with JSON body

It's common to send request body with a `POST` request. The most common body type is a JSON object:

```cs
using LightJson;

var response = Http.Post(
    "http://test.com/do-something",
    new JsonObject {
        ["foo"] = "bar",
        ["baz"] = 42
    }
);
```

> **Info:** Methods `PUT`, `PATCH` and `DELETE` also support this approach.



#### GET request with query parameters

Instead of appending the query string directly to the URL (authough you still can), you can pass it as a dictionary via the second argument:

```cs
var response = Http.Get(
    "http://test.com/query-something",
    new Dictionary<string, string> {
        ["foo"] = "bar",
        ["baz"] = "42"
    }
);
```

> **Info:** Giving a `Dictionary<string, string>` as a second argument to other methods (`POST`, `PUT`, `PATCH` and `DELETE`) will send it as an form URL encoded body.


#### Specifying request body in advance

You can specify the request body before you send the request:

```cs
// JSON
var response = Http.WithJsonBody(
    new JsonObject {
        ["foo"] = "bar",
        ["baz"] = 42
    }
).Post("http://test.com/do-something");

// Form URL encoded
var response = Http.WithFormBody(
    new Dictionary<string, string> {
        ["foo"] = "bar",
        ["baz"] = "42"
    }
).Post("http://test.com/do-something");
```


#### Sending raw `HttpContent`

If you want to send a different body, you can always fallback to the [.NET HttpContent class](https://docs.microsoft.com/en-us/dotnet/api/system.net.http.httpcontent):

```cs
using System.Net.Http;

var response = Http.WithBody(
    new StringContent(
        "Hello!",
        Encoding.UTF8,
        "text/plain"
    )
).Post("http://test.com/do-something");
```


<a name="headers"></a>
### Headers

You can add additional headers to the request using the `WithHeaders` method:

```cs
var response = Http.WithHeaders(
    new Dictionary<string, string>(
        ["X-First"] = "foo",
        ["X-Second"] = "bar"
    )
).Post("http://test.com/do-something");
```


<a name="authentication"></a>
### Authentication

You may add authentication information via the HTTP basic authentication scheme or the OAuth 2.0 bearer token scheme:

```cs
var response = Http
    .WithBasicAuth("username", "password")
    .Post(...);

var response = Http
    .WithToken("token")
    .Post(...);
```


<a name="error-handling"></a>
### Error handling

Unisave HTTP client does not throw exceptions on client or server errors (`4xx` or `5xx` status codes).
You can check for the error using the following properties:

```cs
response.Status; // int
response.IsOk; // true when status is 200
response.IsSuccessful; // true when status is 2xx
response.Failed; // true when status is 4xx or 5xx
response.IsClientError; // true when status is 4xx
response.IsServerError; // true when status is 5xx
```

If you, however, do want to throw an exception in such case, you can call the `Throw` method. This method will, of course, not do anything if the request was successful.

```cs
var response = Http.Post(...);

response.Throw();

return response["foo"]["bar"];
```

The method will throw an instance of `System.Net.HttpRequestException`.

The method also returns the response itself so it can be chained:

```cs
return Http.Post(...).Throw().Json()["foo"]["bar"];
```


<a name="the-underlying-net-instance"></a>
### The underlying .NET instance

Whenever you have an instance of a request or a response, you can access the respective `HttpRequestMessage` and `HttpResponseMessage` via the `.Original` property:

```cs
var response = Http.Get(...);

response.Original.StatusCode; // 404
response.Original.ReasonPhrase; // "Not found"
```


<a name="testing"></a>
## Testing


TODO
