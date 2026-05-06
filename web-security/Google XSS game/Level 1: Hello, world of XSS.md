# [Level 1 Hello, world of XSS](https://xss-game.appspot.com/level1)


## Observations

When you enter a search term like `hello`, the page responds with:

```
Sorry, no results were found for hello. Try again.
```

The URL becomes:

```
https://xss-game.appspot.com/level1/frame?query=hello
```

This suggests a **Reflected XSS vulnerability**, because the user input (`hello`) is reflected directly in the HTML response without proper sanitization.


## What is Reflected XSS?

**Reflected XSS** happens when user input (e.g., from a search box or URL) is shown on the page **without being properly encoded or filtered**, allowing attackers to inject and execute JavaScript.


## Solution
Make the URL like this:

```
https://xss-game.appspot.com/level1/frame?query=%3Cscript%3Ealert(1)%3C/script%3E
```

This will cause the browser to execute the JavaScript `alert(1)` — confirming the vulnerability.

* The input from `query=` is inserted directly into the page’s HTML.
* Since it’s not properly escaped or sanitized, the `<script>` tag is interpreted as real JavaScript by the browser.

The problem is solved.
