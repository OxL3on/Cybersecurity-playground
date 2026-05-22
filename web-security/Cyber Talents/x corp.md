The page used obfuscated JavaScript that sent user input to a hidden PHP endpoint and executed the server response using `eval()`:

```javascript id="q4m8vx"
eval(a)
```

This indicated a possible XSS vulnerability.

After testing different payloads, I successfully triggered JavaScript execution using:

```txt id="r7q2va"
admin'onload=alert(1)
```

The payload executed in the browser, confirming a reflected XSS vulnerability caused by unsafe handling of user input together with `eval()`.
