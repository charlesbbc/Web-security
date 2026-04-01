# Web caches

User key for the requests, if it matches then it serves the cached copy.


String rules in URL path:

* Static file extension: `.css` or `.js` for example
* Static directory: parth that start with directory like `/static` or `/assets`
* File name: `robots.txt` or `favicon.ico`

## Web cache deception attack
1. Identify endpoint with sensitive info. Focus `GET`, `HEAD` or `OPTIONS`, others usually not cached.
2. Identify discrepancy how the cache and origin server parse the URL path. Discrepancies:
   * Mapping URL to resources
   * Proces delimiter chars
   * normalize paths
4. Craft malicious URL that uses the discrepancy to trick cache into storing a dynamic response. When victim accesses, the response is stored in cache. With Burp, can send a request to the same URL to fecth cached response with victim's data. Avoid this in the browser, as some apps redirect users without session or invalidate local data, which could hide a vulnerability.

## Using a cache buster

Each request should have a key. Both URL path and query params are typically included in the cached key.

Can change the key adding a query string to the path and automate with Param Miner.

Param Miner -> Settings -> Add dynamic cachebuster

Can view this string on the Logger tab.

## Detecting cached responses:

Responses that may indicate being cached:

`X-Cache` header says if it was served from cache:

* `X-Cache: hit` - served from cache
* `X-Cache: miss` - not contain a response for the request's key. Most will cache, can send the request again and check if value updates to hit.
* `X-Cache: dynamic` - generally means not cached
* `X-Cache: refresh` - cached is outdated, needs refresh

`cache-control` header may include directive for caching like, `public` or `max-age > 0`. It only says it's cacheable, not necessarily.

If notice a big diference in response time also may indicate cache.

If a cache and a server disagree on how to read a URL, an attacker can trick the cache into storing a private page by giving it a fake "static" file extension.
