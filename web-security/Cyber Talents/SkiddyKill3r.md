The challenge started with a search functionality.
Searching for the username:

```txt id="m4q8vx"
Momen
```

revealed a hint about parameter brute-forcing.

After fuzzing parameters, I discovered:

```txt id="r7q2va"
?show=True
```

which exposed the PHP source code using `show_source()`.

The source contained several conditions:

* special `Referer` header check
* MD5/type juggling vulnerability
* hidden hint parameter
* User-Agent validation

To get the first flag part, I sent:

```http id="k1v9mc"
Referer: http://cyberguy
```

The second part used PHP magic hash/type juggling with:

```txt id="p5m3zu"
flag=240610708
flag1=QNKCDZO
```

Both values generate MD5 hashes starting with `0e`, allowing the loose comparison to bypass the check.

Next, using:

```txt id="t8q4ra"
?flag=HiNt
Cookie: flag=True
```

revealed a hint pointing to `robots.txt`.

Inside `robots.txt`, another hidden endpoint was disclosed.
Finally, sending the special User-Agent:

```txt id="x2v7pk"
G3t_My_Fl@g_N0w()
```

to:

```txt id="y5m1qn"
/user_check.php
```

returned the final flag:

```txt id="z9q3wa"
0xL4ugh{H3r0_I5_You0_F0r_N0w}
```
