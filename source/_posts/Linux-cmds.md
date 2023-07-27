---
title: Linux cmds
date: 2023-06-04 19:18:25
tags: Linux
category: commands
---

# Basic linux cmds

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
## Syscall related commands

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