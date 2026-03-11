# Prototype Pollution basics

In client-side JavaScript, commonly leads to DOM XSS, while server-side could lead to RCE.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
const user =  {
    username: "wiener",
    userId: 01234,
    isAdmin: false
}

user.username
user['userId']
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Can have functions (in this case known as method):
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
const user =  {
    username: "wiener",
    userId: 01234,
    exampleMethod: function(){
        // do something
    }
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
## Prototype in JavaScript and inheritance

Object linked to another object, is its prototype. There are some built-in automatically assigned.

strings -> String.prototype

![Image](/assets/img/prototype_of.png)

String.prototype -> toLowerCase() method

All strings have this method.

There is an Object prototype also.

Inheritance works so that it first checks the objects property and if it does not find it, then looks for the prototype one.

`__proto__`   getter & setter for prototype object

`username.__proto__`

`username['__proto__']`

`username.__proto__`                        // String.prototype

`username.__proto__.__proto__`              // Object.prototype

`username.__proto__.__proto__.__proto__`    // null

Before `trim()` in strings, developers modified String.prototype with custom implementation.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
String.prototype.removeWhitespace = function(){
    // remove leading and trailing whitespace
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Then all strings have access to the method:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
let searchTerm = "  example ";
searchTerm.removeWhitespace();
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
