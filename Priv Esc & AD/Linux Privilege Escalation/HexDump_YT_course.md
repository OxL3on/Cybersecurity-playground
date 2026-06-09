
`ssh -o "UserKnownHostsFile=/dev/null" user@ip` - Want to save fingerprint or not

## Difference between Terminal, TTY and Shell

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


## Basic Commands

- `whoami` : gives current username
- `id` : userid, groupid, which group you belong to
- `hostname` : name of your host
- `pwd` : print working directory
- `env` : variables that control the program in the shell or also the other programs
- `which commandName` : the path within a file system for that particular program


## Relative and Absolute Paths

- Absolute path : Always starts with root of the file system
- Relative path : Dont start from the root but start from the current working directory


## File System Commands

- `pwd` : print working directory
- `cd` : change directory
- `ls` : list 
- `ls -al` : Show hidden files
- `mv` : move files
- `cp` : copy files
- `rm` : remove files
- `echo "hello1" > text1.txt` : create a file with content hello1


## Resources Management Commands

- `fdisk -l` : How many devices are connected to our computer
- `df -h` : How much space if taken or left
- `du -h` : shows how much spack a certain directory holds
- `ps` : Process
- `ps aux` : system processes
- `ps -axjf` : Hierarchy of the process
- `ip address , ip a` : network interfaces
- `netstat -ltp` : Display all the TCP listening ports, displaying PID/program names and resolve names with IP address on that machine



## User Management Commands

