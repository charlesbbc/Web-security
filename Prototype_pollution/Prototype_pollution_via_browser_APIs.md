# Prototype pollution via browser APIs

Gadgets in the JS APIs

## Prototype pollution via fetch()

HTTP requests. 2 args (URL, option for parts like method, headers, body params, etc)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
fetch('https://normal-website.com/my-account/change-email', {
    method: 'POST',
    body: 'user=carlos&email=carlos%40ginandjuice.shop'
})
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
If an attacker can find a source, they could pollute Object.prototype with their own headers property.

Code vulnerable to DOM XSS via prototype pollution:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
fetch('/my-products.json',{method:"GET"})
    .then((response) => response.json())
    .then((data) => {
        let username = data['x-username'];
        let message = document.querySelector('.message');
        if(username) {
            message.innerHTML = `My products. Logged in as <b>${username}</b>`;
        }
        let productList = document.querySelector('ul.products');
        for(let product of data) {
            let product = document.createElement('li');
            product.append(product.name);
            productList.append(product);
        }
    })
    .catch(console.error);
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
An attacker could pollute Object.prototype with a headers property containing a malicious x-username header:

`?__proto__[headers][x-username]=<img/src/onerror=alert(1)>`

## Prototype pollution via Object.defineProperty()

Developers may block potential gadgets by `Object.defineProperty()`.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Object.defineProperty(vulnerableObject, 'gadgetProperty', {
    configurable: false,
    writable: false
})
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
But it is flawed. Just like `fetch()`, it accepts an option object. Developers can use descriptor object to initialize the defined property. But if it is just to protect agains prototype poluttion, better not.

An attacker could bypass it polluting the `Object.prototype` with malicious value property. If this is inherited by the descriptor object passed to `Object.defineProperty()`, the attacker value may be assigned to the gadget property.

Example:

`/?__proto__[value]=data:,alert(1)`
