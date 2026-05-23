The application allowed user-controlled input to be stored and rendered directly inside the page without sanitization.

I tested the username field with the following payload:

```html id="m4q8vx"
<img src=x onerror=alert(1)>
```

After saving the profile, the payload executed successfully in the browser, confirming a Stored XSS vulnerability.
