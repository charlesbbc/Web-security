# Preventing prototype pollution vulnerabilities

## Sanitizing property keys

Sanitizing before merging to objects. Can prevent keys such as `__proto__`

Using allowlist, when not possible use blocklist (can be tricky). Can be bypassed with constructor or obfuscation.

## Preventing changes to prototype objects

Prevent objects of being changed.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Object.freeze()
Object.freeze(Object.prototype);
Object.seal()  //is similar but allows changes to values of existing properties.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
## Preventing an object from inheriting properties

Can set object prototype using `Object.create()`. Can use any object as prototype or use null prototype not to inherit properties.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
let myObject = Object.create(null);
Object.getPrototypeOf(myObject);    // null
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

## Using safer alternatives where possible

Objects with built-in protections. Ex, whenm defining options object, can use a Map instead. Can inherit malicious properties but the have a built-in `get()` that only returns properties defined on the Map.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Object.prototype.evil = 'polluted';
let options = new Map();
options.set('transport_url', 'https://normal-website.com');

options.evil;                    // 'polluted'
options.get('evil');             // undefined
options.get('transport_url');    // 'https://normal-website.com'
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
A `set` is another alternative to store key:value pairs. Like `maps`, only return properties defined on the object itself.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Object.prototype.evil = 'polluted';
let options = new Set();
options.add('safe');

options.evil;           // 'polluted';
option.has('evil');     // false
options.has('safe');    // true
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
