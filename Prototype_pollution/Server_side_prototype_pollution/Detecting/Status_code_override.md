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
