# [Level 2: Persistence is key](https://xss-game.appspot.com/level2)

### Observation:

* There's a text field to "share a status."
* When you type something and click **Share**, it **stays** on the page — this hints at **stored XSS**.
* I first tested with:

  ```html
  <h1>hello</h1>
  ```

  After sharing, the text appeared in **large size**, meaning **HTML is not sanitized**.


### Exploit:

To trigger an alert popup using JavaScript, use:

```html
<img src="" onerror="alert()">
```

* Paste this into the status field.
* Click **Share status**.
* The payload gets stored, and when the page reloads or is viewed, the `onerror` event runs `alert()`.

* **Lab solved** 

