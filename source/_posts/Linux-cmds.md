---
title: Linux cmds
date: 2023-06-04 19:18:25
tags: Linux
category: commands
---

# Basic linux cmds


## Index
1. [System check realted commands](#system-check-related-commands)
2. [Input/output related commands](#inputoutput-related-commands)
3. [Shell and bash scripting](#shell-and-bash-scripting-related)


## System check related commands

### Resource check commands

- Check CPU info

``` bash
    cat /proc/cpuinfo
```

or 

``` bash
    ls cpu
```

- Check memory

``` bash
    grep MemTotal /proc/meminfo
```

or 
``` bash
    free -m # show in mbyte
    free -h # human-readable forms
```

- Check Virtual memory

``` bash
    grep VmallocTotal /procm/meminfo 
```

- check block device
``` bash
    lsblk
```
- display disk partition status

``` bash
    fdisk -l
```

- Check devices

``` bash
    ls -la /sys/devices
```

- Check folder size
``` bash
    du -xhs /<PATH>
```
### Network related commands
``` bash
    netstat -t -u -n -p -l # tcp | udp | numeric(do not resolve hostnames) | programs | listening

    Active Internet connections (only servers)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
    tcp        0      0 localhost:43619         0.0.0.0:*               LISTEN      13331/node
    tcp        0      0 127.0.0.53:domain       0.0.0.0:*               LISTEN      1860/systemd-resolv
    tcp        0      0 localhost:34781         0.0.0.0:*               LISTEN      248/containerd
    tcp        0      0 localhost:32809         0.0.0.0:*               LISTEN      13259/node
    udp        0      0 127.0.0.53:domain       0.0.0.0:*                           1860/systemd-resolv
    udp        0      0 localhost:323           0.0.0.0:*                           -
    udp6       0      0 ip6-localhost:323       [::]:*                              -
```

### Process related commands 
- List current process and show in tree form
``` bash
    pstree
```
### Syscall related commands

- Follow syscall

``` bash
    strace + syscall_cmd
```

> -c summary only

- Check kernel machine and kernel release

``` bash
    uname -srm
```


## Input/output related commands
- Translate with input/outs. tr command.
``` bash
    tr [input] [output]

# Examples
# Input some texts into tr command then translate to all CAPS

    tr [a-z] [A-Z] < input.txt >output.txt

# delete numbers in the given input
    echo "My UID is $UID" | tr -cd "[:digit:]\n"
```
for detailed SETS, need to check with the `tr --help` command

### Locate file/folders
``` bash
    locate # Find all files available with keyword 
    whereis # find binary file, config file and man files
    which # find the position of the command executed
```

### Find command 
- Fo example, to find avaiable modules in Linux.
``` bash
    find /lib/modules/$(umame -r) -type f -name '*.ko*'
    # find "in directory - type file or else -name '' with anything related to .ko
    # It didn't work on my WSL ubuntu
```

### To list modules that loaded by kernel
``` bash
    lsmod
```

## Shell and bash scripting related

### terminal and shell
- solve problem of ```/bin/bash^m bad interpreter no such file or directory```

``` bash
    sed -i 's/\r//' <filename>
```
- check terminal configuration
```bash
    infocmp
#       Reconstructed via infocmp from file: /lib/terminfo/x/xterm-256color
xterm-256color|xterm with 256 colors,
        am, bce, ccc, km, mc5i, mir, msgr, npc, xenl,
        colors#0x100, cols#80, it#8, lines#24, pairs#0x10000,
        acsc=``aaffggiijjkkllmmnnooppqqrrssttuuvvwwxxyyzz{{||}}~~,
```

In general, shell uses File Descriptors for input and output
- stdin (FD 0)
- stdout (FD 1)
- stderr (FD 2)

Here is some redirect examples:
``` bash
    curl https://sth.com &> /dev/null # no output generated 
    
    curl https://sth.com 1> output 2> curl_status # generate std output and status of the execution of curl seperatelly.

    curl https://sth.com 1>&2 file # 2 kind of outputs in one file.

```

### Variables

There are *Shell variables* and *Environmental Variables* in Linux.
Shell variables only exist in local shell process and are not inherited by subprocesses. Env variables are shell-wide.

``` bash
    MY_VAR=42 # Shell variable
    export MY_GLOBAL_VAR=250 # Global variable
    
    $ set | grep MY
    MY_GLOBAL_VAR=250
    MY_VAR=42
    $ env | grep MY
    MY_GLOBAL_VAR=250
    $ bash
    $ set | grep MY  # Shell variable only exists in current shell.
    MY_GLOBAL_VAR=250
```
Full environmental variable list is under [GNU bash manual](https://www.gnu.org/software/bash/manual/html_node/Bash-Variables.html).

### Exit status

The result of the execution of commands are determined by the exit status code from 0-255, in which 0 means normal. To querry the exit status, use `echo &?` 