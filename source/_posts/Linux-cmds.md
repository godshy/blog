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
4. [Security and cert](#security-and-certificate-generation)
5. [VM management](#kvm-related)

## System check related commands




### Often used basic commands and their variations

- 'cat' command and its successor 'bat'

### Log check commands 

- use journalctl to check service logs

``` bash
    journalctl -f -u <someservice.service>

```

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

- check logical volume, physical volume and volume group 
``` bash
    sudo lvscan # list logical volumes
    sudo vgs # list volume group
    sudo pvdisplay # list physical volume
```

- check I/O status
``` bash
    iostat -z -- human
```

- use lsof command to follow process

``` bash
    # check port usage 
    sudo lsof -i TCP:1-1024 # check private port usage
    sudo lsof -p <pid> -i UDP # check process's udp usage
    # check process
    sudo lsof -p <pid> # follow process's activity

```

- check uptime
``` bash
    uptime
```

- chec khow long an operation takes
``` bash
    time (the command)
    # for example
    time (ls -lf)
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

``` bash
grep -v "rem_address" /proc/net/tcp  | awk  '{x=strtonum("0x"substr($3,index($3,":")-2,2)); for (i=5; i>0; i-=2) x = x"."strtonum("0x"substr($3,i,2))}{print x":"strtonum("0x"substr($3,index($3,":")+1,4))}'
# If nestat is not there, we can check connection using this instead.
```


- check ip link conditions arps
``` bash
    ip link show
    ip addr show

    # check arp
    arp
    # check ip neighbors/ arp neighbors
    ip neigh

    # check net adaptor related 
    iw dev <nic_name> info
```

- check if ip forwarding is enabled
``` bash
    cat /proc/sys/net/ipv4/ip_forward # 1 for enable
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

- use  -c summarize the overview of the command

- Check kernel machine and kernel release

``` bash
    uname -srm
```

- Check capability (Advanced access control)
``` bash
    capsh --print
    grep Cap /proc/$$/status # capabilities for the current process
```

### cgroup and namespace related commands

``` bash
    sudo lsns # check namespace
    systemctl status # check cgroup
```
### Directory sync commands

``` bash
    rsync <source_ip> <user_name>@<target_ip>:/dir
    #a -- for archive keep file group, version etc
    #z -- for compress
    #p -- keep permissions
    #q -- quiet mode
    #v -- verbose
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

### File content management

- Create a file using here document. "EOF"
``` bash
    cat << 'EOF' > /some_dir/file_name
    # ENDLINE with EOF
```

- Compare to files side by side
``` bash
    diff -y file1 file2
```
- __To see a constantly growing file__
``` bash
    tail -f sth.log
```
### Find command 
- For example, to find available modules in Linux.
``` bash
    find /lib/modules/$(umame -r) -type f -name '*.ko*'
    # find "in directory - type file or else -name '' with anything related to .ko
    # It didn't work on my WSL ubuntu

#'find' command and its better version

    # example of using find and rg to find yaml files that contains "sample"
    find . -type f -name "*.yaml" -exec grep "sample" '{}' \; -print
    rg -t "yaml" sample

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

The result of the execution of commands are determined by the exit status code from 0-255, in which 0 means normal. To query the exit status, use ` echo &? ` 

### Job control 

We can run commands in background, and then take it back to the foreground. Also use ctrl+z is able to put command into background.

``` bash
    <some command> & # use & to run commands in backend
    fg # bring command back to frontend
    jobs # watch current jobs in the backend

    disown
    nohup # disown and nohup
```

Terminal multiplexer, is also able to maintain multiple tasks. 


### Json processing command jq

For example, we have a following json file.
``` json
{
  "menu": {
    "id": "file",
    "value": "File",
    "popup": {
      "menuitem": [
        {
          "value": "New",
          "onclick": "CreateNewDoc()"
        },
        {
          "value": "Open",
          "onclick": "OpenDoc()"
        }
      ]
    }
  }
}
```
dot '.' followed by the object is used to access the value of jq.      
``` bash
    # this shows the whole file of test.json
    jq '.' test.json
    # use multiple dots to show the nested proterty
    jq '.menu.value, .menu.id' test.json
```
```
"File"
"file"
```
It is able to use <code>[]</code> to show lists in json file.
``` bash
    jq '.menu.popup.menuitem' test.json | jq '.[]'
```
``` json
{
  "value": "New",
  "onclick": "CreateNewDoc()"
}
{
  "value": "Open",
  "onclick": "OpenDoc()"
}   
```
Also, it is able to use index in the <code>[]</code> operator. 

``` bash
     jq '.menu.popup.menuitem' test.json | jq '.[0].value'
```
```
"New"  
```

### Time display

``` bash
    # show time in linux epoch
    date +%s

    # linux time to human readable time
    date -d @1707912602 '+%m/%d/%Y :%H:%M:%S'
```

## Security and certificate generation

Generate keys for CA.

``` bash
    # generate private key
    openssl genrsa -out ca.key 2048 
    # create CSR
    openssl req -new -key ca.key -subj "/CN=" -out ca.csr
    # sign the CSR
    openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
```

Generate keys cert for client
``` bash
    openssl genrsa -out admin.key 2048
    openssl req -new -key admin.key -subj "/CN=kube-admin/O=system:masters" -out admin.csr
    # Sign the csr using CA's private key
    openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt
    

```

Check certificate details
``` bash
    openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout
```

## KVM related

- check vm list
``` bash
    virsh list --all
```
- login into a vm
``` bash
    virsh console <vm-name>
```
- to logout from vm, use ```ctrl+5```

- Check VM IP nad net stack

``` bash
    virsh net-list # show network name
    virsh net-info <net_name> # default named default
    virsh net-dhcp-leases <net_name>

```

