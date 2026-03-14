## Detecting server-side prototype pollution without polluted property reflection

Most of the time with server pollution, will not get a reflected response.

Can try properties that potentially match server config.

Techniques:

* Status code override
* JSON spaces override
* Charset override

## Status code override

Server side frameworks like Express allow to set custom statuses. If error, then response may issue generic answer with en error object.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
HTTP/1.1 200 OK
...
{
    "error": {
        "success": false,
        "status": 401,
        "message": "You do not have permission to access this resource."
    }
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Node's http-errors contains  the following function:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
function createError () {
    //...
    if (type === 'object' && arg instanceof Error) {
        err = arg
        status = err.status || err.statusCode || status
    } else if (type === 'number' && i === 0) {
    //...
    if (typeof status !== 'number' ||
    (!statuses.message[status] && (status < 400 || status >= 600))) {
        status = 500
    }
    //...
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~    
If the developers haven't set a status property for the error, you can potentially use this to probe for prototype pollution as follows:

1. Find a way to trigger an error response and take note of the default status code.
2. Try polluting the prototype with your own status property. Be sure to use an obscure status code that is unlikely to be issued for any other reason.
3. Trigger the error response again and check whether you've successfully overridden the status code.

*Must choose a status code in the 400-599 range, otherwise Node defaults a 500.

## JSON spaces override

Express framework has json spaces, to configure number of spaces to indent JSON data in the response. If left with the default, it can have pollution via the prototype chain.

If got access to a JSON response, then can try polluting the prototype with your own json spaces property, the resissue the request to see if indentation increases (same to decrease).

It has been fixed in Express 4.17.4, but websites that haven't upgraded may still be vulnerable.

To see it in burp, need to put in raw, to correctly see indentation.

## Charset override

Express servers have “middleware” to process requests before giving to the correct handler. `body-parser` on requests to generate `req.body`. This contains another gadet for prototype pollution.

The following code, passes object into the `read()` function, used to read in request body to parse. There is encoding, gets `getCharset(req)` or defaults `UTF-8`. 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
var charset = getCharset(req) or 'utf-8'
function getCharset (req) {
    try {
        return (contentType.parse(req).parameters.charset || '').toLowerCase()
    } catch (e) {
        return undefined
    }
}
read(req, res, next, parse, debug, {
    encoding: charset,
    inflate: inflate,
    limit: limit,
    verify: verify
})
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
### Example:

If can find an object with properties visible in responses, can use it to probe for sources. Ex. using `UTF-7` and JSON source:

1. Add arbitrary `UTF-7` string foo in `UTF-7` is +AGYAbwBv-.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
{
    "sessionId":"0123456789",
    "username":"wiener",
    "role":"+AGYAbwBv-"
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
2. Send request, servers won't use `UTF-7` by default, so the strings should appear in the encoded form.

3. Pollute with a content-type to specify `UTF-7`
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
{
    "sessionId":"0123456789",
    "username":"wiener",
    "role":"default",
    "__proto__":{
        "content-type": "application/json; charset=utf-7"
    }
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
4. Repeat the request and the response should contain the decoded `UTF-7` string.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
{
    "sessionId":"0123456789",
    "username":"wiener",
    "role":"foo"
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
There is a bug on Node's `_http_incoming` module, even if `Conten-Type` header includes the charset.

`_addHeaderLine()` checks that no property already exists before trasnfering properties to an `IncomingMessage` object.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
IncomingMessage.prototype._addHeaderLine = _addHeaderLine;
function _addHeaderLine(field, value, dest) {
    // ...
    } else if (dest[field] === undefined) {
        // Drop duplicates
        dest[field] = value;
    }
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
It includes properties inherited from prototype chain, then can pollute prototype with the content-type property.
