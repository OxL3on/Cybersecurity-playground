The newsletter form accepted unsanitized user input, which indicated a possible Command Injection vulnerability.

Using BurpSuite, I intercepted the request and modified the `email` parameter:

```txt id="v0f1re"
email=test@test.com;ls||
```

The injected `ls` command listed files in the server directory and revealed the backup filename:

```txt id="m2y8kx"
hgdr64.backup.tar.gz
```
