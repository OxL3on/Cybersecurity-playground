The application responded with:

```txt id="m4q8vx"
SORRY , toka only talks German :)
```

This hinted that the server was checking the `Accept-Language` HTTP header.

I modified the request header to German:

```http id="r7q2va"
Accept-Language: de
```

After sending the request with the German language header, the application returned the flag.
