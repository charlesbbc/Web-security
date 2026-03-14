## JSON spaces override

Express framework has json spaces, to configure number of spaces to indent JSON data in the response. If left with the default, it can have pollution via the prototype chain.

If got access to a JSON response, then can try polluting the prototype with your own json spaces property, the resissue the request to see if indentation increases (same to decrease).

It has been fixed in Express 4.17.4, but websites that haven't upgraded may still be vulnerable.

To see it in burp, need to put in raw, to correctly see indentation.
