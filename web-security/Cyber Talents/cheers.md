The challenge leaked PHP notices revealing undefined GET parameters: First its gave `Undefined index: welcome` this parameter and then If I use `/?welcome=anything` It gave `Undefined index: gimme_flag` this parameter


This indicated that the application expected user-controlled parameters in the URL.

I added both parameters manually:

```txt id="r8v1mx"
index.php?welcome=h&gimme_flag=cheers
```

After supplying the required parameters, the application returned the flag.
