The login page was vulnerable to SQL Injection because user input was directly included in the SQL query without proper sanitization.

I tested the username field with the following payload:

```sql id="p3x8mv"
username = admin' OR 1=1#
password = anything
```

This works and I got the flag.

(If you view the page source You'll get creds like bob:password but if you login with those you wont get the flag)
