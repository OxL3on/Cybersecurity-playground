The challenge exposed the `.git` directory, which allowed the entire source code of the web application to be downloaded.

I used `git-dumper` to retrieve the repository:

```bash id="k2m8vx"
git-dumper http://TARGET/.git/ dump
```

After inspecting the dumped files, I found `flag.php` containing the secret value:

```php id="f7q1pa"
$secret_key = 'be607453caada6a05d00c0ea0057f733';
```

