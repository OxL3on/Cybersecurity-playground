# Easy Win! Writeup

The login functionality was implemented completely in client-side JavaScript and obfuscated to hide the credentials.

After inspecting and deobfuscating the script, the hardcoded credentials were revealed:

```javascript id="j8f2wp"
email = "admin@mail.com"
password = "password"
```

eThe challenge can be solved by either:

* logging in with the discovered credentials


