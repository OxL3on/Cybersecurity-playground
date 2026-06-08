
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
