# Handling a Request

This lesson describes how to implement your own custom url
Creating a callback url is as  easy as implementing a script on your web server that handles HTTP POST requests.

## A Few Things To Note About The Callback

For cases where you do need to implement a custom request, this is all you need
to note:

- Every request we send you will contain a sessionId, each sesssion is will last for 3 mins(180 seconds), after then a session is terminated.
- You will need to let us  know whether the session is complete or not. If the session is ongoing, please begin your response with CON. If this is the last response for that session, begin your response with END.
- If we get a HTTP error response (Code 40X) from your script, or a malformed response (does not begin with CON or END), we will terminate the session gracefully.
- Your Menu should not contain special characters as the telcos are unable to render such content, which could lead to disruptions in accessing your USSD services.

### API parameters

The API makes a `HTTP POST` request to your server with the parameters shown below, you can read the data sent from the form fields of the request. Content-Type: `application/x-www-form-urlencoded`. This request is made when the user interacts with your whatsappbot.

## Parameters

**sessionID**`String`

A unique value generated when the session starts and sent every time a mobile subscriber response has been received.

[**SetUp  A Simple callbackURL in JavaScript**]

Learn how to create a simple callbakURl using javascript.

[**SetUp  A Simple callbackURL in Java**]
Learn how to create a simple callbakURl using javascript.

[**SetUp  A Simple callbackURL in Java**]

Learn how to create a simple callbakURl using javascript.

*Kotlin*

```kotlin
override fun parseNetworkResponse(response: NetworkResponse?): Response<T> {
    return try {
        val json = String(
                response?.data ?: ByteArray(0),
                Charset.forName(HttpHeaderParser.parseCharset(response?.headers)))
        Response.success(
                gson.fromJson(json, clazz),
                HttpHeaderParser.parseCacheHeaders(response))
    }
    // handle errors
}
```

*Java*

```java
@Override
protected Response<T> parseNetworkResponse(NetworkResponse response) {
    try {
        String json = new String(response.data,
                HttpHeaderParser.parseCharset(response.headers));
        return Response.success(gson.fromJson(json, clazz),
                HttpHeaderParser.parseCacheHeaders(response));
    }
    // handle errors
}
```

Note the following:

- `parseNetworkResponse()` takes as its parameter a `NetworkResponse`, which
  contains the response payload as a byte[], HTTP status code, and response headers.
- Your implementation must return a `Response<T>`, which contains your typed
  response object and cache metadata or an error, such as in the case of a parse failure.

If your protocol has non-standard cache semantics, you can build a `Cache.Entry`
yourself, but most requests are fine with something like this:

*Kotlin*

```kotlin
return Response.success(myDecodedObject,
        HttpHeaderParser.parseCacheHeaders(response))
```

*Java*

```java
return Response.success(myDecodedObject,
        HttpHeaderParser.parseCacheHeaders(response));
```

Volley calls `parseNetworkResponse()` from a worker thread. This ensures that
expensive parsing operations, such as decoding a JPEG into a Bitmap, don't block the UI
thread.

### deliverResponse

Volley calls you back on the main thread with the object you returned in
`parseNetworkResponse()`. Most requests invoke a callback interface here,
for example:

*Kotlin*

```kotlin
override fun deliverResponse(response: T) = listener.onResponse(response)
```

*Java*

```java
protected void deliverResponse(T response) {
    listener.onResponse(response);
}
```

## Example: GsonRequest

[Gson](https://github.com/google/gson) is a library for converting
Java objects to and from JSON using reflection. You can define Java objects that have the
same names as their corresponding JSON keys, pass Gson the class object, and Gson will fill
in the fields for you. Here's a complete implementation of a Volley request that uses
Gson for parsing:

*Kotlin*

```kotlin
/**
 * Make a GET request and return a parsed object from JSON.
 *
 * @param url URL of the request to make
 * @param clazz Relevant class object, for Gson's reflection
 * @param headers Map of request headers
 */
class GsonRequest<T>(
        url: String,
        private val clazz: Class<T>,
        private val headers: MutableMap<String, String>?,
        private val listener: Response.Listener<T>,
        errorListener: Response.ErrorListener
) : Request<T>(Method.GET, url, errorListener) {
    private val gson = Gson()

    override fun getHeaders(): MutableMap<String, String> = headers ?: super.getHeaders()

    override fun deliverResponse(response: T) = listener.onResponse(response)

    override fun parseNetworkResponse(response: NetworkResponse?): Response<T> {
        return try {
            val json = String(
                    response?.data ?: ByteArray(0),
                    Charset.forName(HttpHeaderParser.parseCharset(response?.headers)))
            Response.success(
                    gson.fromJson(json, clazz),
                    HttpHeaderParser.parseCacheHeaders(response))
        } catch (e: UnsupportedEncodingException) {
            Response.error(ParseError(e))
        } catch (e: JsonSyntaxException) {
            Response.error(ParseError(e))
        }
    }
}
```

*Java*

```java
public class GsonRequest<T> extends Request<T> {
    private final Gson gson = new Gson();
    private final Class<T> clazz;
    private final Map<String, String> headers;
    private final Listener<T> listener;

    /**
     * Make a GET request and return a parsed object from JSON.
     *
     * @param url URL of the request to make
     * @param clazz Relevant class object, for Gson's reflection
     * @param headers Map of request headers
     */
    public GsonRequest(String url, Class<T> clazz, Map<String, String> headers,
            Listener<T> listener, ErrorListener errorListener) {
        super(Method.GET, url, errorListener);
        this.clazz = clazz;
        this.headers = headers;
        this.listener = listener;
    }

    @Override
    public Map<String, String> getHeaders() throws AuthFailureError {
        return headers != null ? headers : super.getHeaders();
    }

    @Override
    protected void deliverResponse(T response) {
        listener.onResponse(response);
    }

    @Override
    protected Response<T> parseNetworkResponse(NetworkResponse response) {
        try {
            String json = new String(
                    response.data,
                    HttpHeaderParser.parseCharset(response.headers));
            return Response.success(
                    gson.fromJson(json, clazz),
                    HttpHeaderParser.parseCacheHeaders(response));
        } catch (UnsupportedEncodingException e) {
            return Response.error(new ParseError(e));
        } catch (JsonSyntaxException e) {
            return Response.error(new ParseError(e));
        }
    }
}
```

Volley provides ready-to-use `JsonArrayRequest` and `JsonArrayObject` classes
if you prefer to take that approach. See [Make a standard request](request.md) for more information.
