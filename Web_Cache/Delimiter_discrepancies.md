# Delimiter discrepancies

Delimeters are boundaries between elements in URLS.

`?` separates path from query string

`/profile;foo.css` Java Spring framework uses `;` to add params, known as matrix variables.

Most other frameworks don't use `;` as delimiter. A cache not using spring would interprest as part of the path, if cache has a rule to store responses ending in `.css`, it might cache and serve profile info as if it were a CSS file.

Ruby on Rails framework uses `.` as delimiter to specify the response format.

`/profile` is HTML formater

`/profile.css` is CSS extension, there isn't a CSS formater, so request isn't accepted and returns error

`/profile.ico` is .ico extension, not recognized by Ruby. Default HTML formatter handles it. If the cache configured to store responses ending in .ico, it would cache profile info as if it were a static file.

### Encoded caracters:

`/profile%00foo.js` OpenLiteSpeed server uses encoded null `%00` as a delimiter.

Most other frameworks respond an error if `%00` is in the URL. But if cache uses Akamai or Fastly, it would interpret `%00` and after.

## Exploiting

Could add a static extension to the path viewed by the cache, not the origin server. Need to identify a char that is used as a delimiter by the origin server but not the cache.

First find delimited chars used by origin server. Add string:

/settings/users/list -> /settings/users/list`aaa`

If response is identical to original, then request is redirected, need to chose another endpoint to test.

Next, add a delimiter between the path and arbitrary srting

/settings/users/list`;aaa`
* If response is identical, then ; is a delimiter and origin server interprets as `/settings/users/list`
* If it matches the response to the path with an arbitrary string, the `;` isn't used and interprets everything as a path `/settings/users/list;aaa`

Once identified delimiters, test if used by cache. Add static extension to the path. If response is cached, indicates:
• cache doesn't use delimiter and interprets full path with static extension
• there is cache rule to store responses ending in `.js`

Test all ASCII chars and a range of extensions including `.css`, `.ico` and `.exe`.

Then can construct the exploit that triggers static extension cache rule. 

Payload: /settings/users/list`;aaa.js`
• The cache interprets the path as: `/settings/users/list;aaa.js`
• The origin server interprets the path as: `/settings/users/list`

Origin returns dynamic profile info stored in cache.

Some delimeters can't be used. Browsers URL encode chars like `{`, `}`, `<`, and `>`, and use `#` to truncate the path. Could use those if cache or origin server decodes these chars.

### Example:

Used the steps before, added random string and got different response, then put delimiter between and found `;` gives same answer. Then added extension, repeated and saw `X-Cache: hit`. Finally launched the exploit to attack user and accessed the URL on our user, getting the API from the attacked user.

`/my-account;aaaa.ico`

`<script>document.location="https://0a38002e042db5d2800e714300950095.web-security-academy.net/my-account;aaaa.ico"</script>`

## Delimiter decoding discrepancies

`/profile%23wcd.css`

The origin decodes `%23` to `#`, to use as delimiter, interpreting `/profile` as path

The cache also uses `#`, but doesn't decode, interpreting `/profile%23wcd.css` as path. If there is a cache for `.css` extension, it will store the response.

`/myaccount%3fwcd.css`

Cache server may store response as there is a rule for `.css` extension. Then decodes `%3f` to `?` and forwars to origin server.

Origin server receives `/myaccount?wcd.css` and uses `?` as a delimiter.

Could exploit decoding discrepancy with an encoded delimiter to add static extension to the path viewed by cache but not by the origin server.

Need to include also non printable chars `%00`, `%0A` and `%09`

## Exploiting static directory cache rules

Cache rules often apply to the following directories
* /static
* /assets
* /scripts
* /images
  
To exploit this also need path traversal. 
