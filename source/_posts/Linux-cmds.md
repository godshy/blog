---
title: Linux cmds
date: 2023-06-04 19:18:25
tags: Linux
category: commands
---

# Basic linux cmds

## System check related commands

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

- Check Virtual memory

``` bash
grep VmallocTotal /procm/meminfo 
```

- Check devices

``` bash
ls -la /sys/devices
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


