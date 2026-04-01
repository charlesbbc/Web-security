# Path mapping discrepancies

Path mapping associates paths to resources on a server. 

2 common types: 
* traditional URL
* RESTful

Traditional URL:

`http://example.com/path/in/filesystem/resource.html`

`/path/in/filesystem/` -> path
  
`resource.html` -> accessed file

REST URLs:

`http://example.com/path/resource/param1/param2`

`/path/resource` -> endpoint

`param1` and `param2` -> params to process the request
	
## Discrepancies in mapping result in cached vuls.

### Example:

`http://example.com/user/123/profile/wcd.css`

REST server could interpret as a request for `/user/123/profile` and answers info for user 123, ignoring `wcd.css`

Traditional URL may see request for file `wcs.css` located in `/profile/user/123` directory. Would serve a CSS file.

## Exploiting path map discrepancies

To test how server maps URL path to resources, add arbitary path segment to the URL of target endpoint. If it has the same data, the origin server abstracts the URL path and ignores segment.

`/api/orders/123 to /api/orders/123/foo`
  
To test cache maps URL to resources, add extension (try different `.css`, `.ico`, `.exe` cause may have rules).

`/api/orders/123/foo to /api/orders/123/foo.js`

Then can craft a URL that returns a dinamic response stored in cache. It has different rules for every other endpoint.

**Extension:** Web Cache Deception Scanner BApp

### Example:

`/my-account` gets API key

`/my-account/whatever.js` maps to the same `/my-account`

A request shows:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
X-Cache: miss  
Cache-Control: max-age=30
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Repeating the request, in less than 30 seconds shows:

`X-Cache: hit`

Attacker can serve:

`<script>document.location="https://0a5f00d503503ec6847ebe6f00320009.web-security-academy.net/my-account/test.js"</script>`

When the user enters the link, and we reopen the link in less than 30 secs from that, we see his APPI key
