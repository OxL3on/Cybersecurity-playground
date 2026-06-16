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

~SUID~ stands for ~Set User ID~.

Binaries that have this bit allow the binary to change user permission during its execution. Specifically, it allows the program to be executed by anyone. Then, during its execution, it will change its permission and take the permissions of the owner of the file.

The executable flag is not (x) but rather (s). This is because the program has the SUID bit set. This means that during its execution, at specific times, the program will change its roles in order to become root. 

- chmod +s/u+s/u-s h.sh (first set +s and then +x for executing)
- sudo -l
- (root) , (ALL) NOPASSWD , (scriptmanager : scriptmanager) NOPASSWD : ALL


## 








