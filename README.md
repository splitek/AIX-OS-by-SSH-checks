# AIX OS (cpu, mem, filesystem) by SSH checks

## Overview

This templates are made for monitoring AIX system (CPU, memory, filesystem) without Zabbix agent. Monitoring through Zabbix SSH checks. You can use it when you are not allowed to install/run any external application in target OS.

* Monitoring done by Zabbix SSH checks
* Commands used to gather the data: uptime, vmstat, lsfs, df

Template files:

[AIX OS by SSH checks.xml](AIX%20OS%20by%20SSH%20checks.xml) for CPU & MEM

[AIX filesystems by SSH checks.xml](AIX%20filesystems%20by%20SSH%20checks.xml) for filesystems

This templates was tested on:

* Zabbix 4.4.x, 5.0.x
* AIX Version 6.1


# Table of Contents:

* [Setup](#setup)
* [How to setup key file authentication](#how-to-setup-key-file-authentication)
* [Used macros](#used-macros)
* [Monitored OS parameters](#monitored-os-parameters)
* [Monitored filesystem parameters](#monitored-filesystem-parameters)

---
## Setup

By default templates are configured to use key file authentication.

**Templates can be used with a user/password pair, but you need to manually switch items `Authentication method` from "Public key" to "Password" and set user/password.** Items to change are:

* `stat data master`,
* `Mounted filesystem discovery`
* `{#FSNAME}: Filesystem data master`

## How to setup key file authentication

Details how to configure Zabbix server/proxy to support key-file based authentication: <https://www.zabbix.com/documentation/current/manual/config/items/itemtypes/ssh_checks#key_file_authentication>

Below is a quick help:

* In server/proxy configuration set full path to a folder where public and private keys will be located, ie. `SSHKeyLocation=/var/lib/zabbix/.ssh` (check permissions for user zabbix to this location).

* If you do not have keys, generate them:

```sh
sudo -u zabbix ssh-keygen -t rsa
```

* Copy key to remote host you want to monitor (replace `username@somehost` with appropriate user and host)

```sh
sudo -u zabbix ssh-copy-id username@somehost
```

* Check login with key-pair

```sh
sudo -u zabbix ssh username@somehost
```

At first login system can ask you about confirmation to insert host into "known_hosts" file - answer "YES". Next login should be straight and fast.

## Used macros

OS template:

| name | default | description |
|-|-|-|
| {$SSH_USER} | username | User name for connection |
| {$CPU.LOAD.BY_SSH} | 5 | Average system load calculated over a given period of time |
| {$CPU.UTIL.CRIT.BY_SSH} | 90 | CPU utilization % |
| {$CPU.WA.BY_SSH} | 30 | Processor idle time during which the system had outstanding disk/NFS I/O request (in %) |

Filesystems template:

| name | default | description |
|-|-|-|
| {$SSH_USER} | username | User name for connection |
| {$VFS.FS.FSNAME.MATCHES} | .+ | This macro is used in filesystems discovery for filesystems selection |
| {$VFS.FS.FSNAME.NOT_MATCHES} | ^(/dev|/sys|/run|/proc|.+/shm$) | This macro is used in filesystems discovery for filesystems selection |
| {$VFS.FS.FSTYPE.MATCHES} | ^(btrfs|ext2|ext3|ext4|reiser|xfs|ffs|ufs|jfs|jfs2|vxfs|hfs|apfs|refs|ntfs|zfs|nfs)$ | This macro is used in filesystems discovery for filesystems selection |
| {$VFS.FS.FSTYPE.NOT_MATCHES} | ^\s$|^procfs$ | This macro is used in filesystems discovery for filesystems selection |
| {$VFS.FS.INODE.PUSED.MAX.CRIT} | 90 | inode critical threshold % |
| {$VFS.FS.INODE.PUSED.MAX.WARN} | 80 | inode warning threshold % |
| {$VFS.FS.PUSED.MAX.CRIT} | 95 | used space critical threshold % |
| {$VFS.FS.PUSED.MAX.WARN} | 92 | used space warning threshold % |

## Monitored OS parameters

#### Load average

| item | key | `uptime` command parameter |
|-|-|-|
| Processor load (1 min average per core) | system.cpu.load.ssh[percpu,avg1] | load average |
| Processor load (5 min average per core) | system.cpu.load.ssh[percpu,avg5] | load average |
| Processor load (15 min average per core) | system.cpu.load.ssh[percpu,avg15] | load average |

#### System configuration

| item | key | `vmstat` command parameter |
|-|-|-|
| Number of logical processors | system.cpu.lnumber.ssh | lcpu - Indicates the number of logical processors |
| Total memory | system.memory.size.ssh[total] | mem - Indicates the amount of memory |
| Entitled capacity | system.ent.ssh | ent - Indicates the entitled capacity. Displays only when the partition is running with shared processor |

#### kthr: Information about kernel thread states

| item | key | `vmstat` command parameter |
|-|-|-|
| Runnable kernel threads (run queue) | system.cpu.kthr.r.ssh | r - Average number of runnable kernel threads over the sampling interval. Runnable threads consist of the threads that are ready but still waiting to run, and the threads that are already running |
| Runnable kernel threads (wait queue) | system.cpu.kthr.b.ssh | b - Average number of kernel threads that are placed in the Virtual Memory Manager (VMM) wait queue (awaiting resource, awaiting input/output) over the sampling interval |

#### Memory: Information about the usage of virtual and real memory. _Virtual pages are considered active if they are accessed. A page is 4096 bytes_

| item | key | `vmstat` command parameter |
|-|-|-|
| Active virtual pages | system.mem.avm.ssh | avm - Active virtual pages |
| Size of the free list | system.mem.fre.ssh | fre - Size of the free list |

#### Page: Information about page faults and paging activity. _This information is averaged over the interval and given in units per second_

| item | key | `vmstat` command parameter |
|-|-|-|
| Pager input/output list | system.mem.page.re.ssh | re - Pager input/output list |
| Pages that are paged in from paging space | system.mem.page.pi.ssh | pi - Pages that are paged in from paging space |
| Pages paged out to paging space | system.mem.page.po.ssh | po - Pages paged out to paging space |
| Pages freed (page replacement) | system.mem.page.fr.ssh | fr - Pages freed (page replacement) |
| Pages that are scanned by page-replacement algorithm | system.mem.page.sr.ssh | sr - Pages that are scanned by page-replacement algorithm |
| Clock cycles by page-replacement algorithm | system.mem.page.cy.ssh | cy - Clock cycles by page-replacement algorithm |

#### Faults: Trap and interrupt rate averages per second over the sampling interval

| item | key | `vmstat` command parameter |
|-|-|-|
| Device interrupts | system.faults.in.ssh | in - Device interrupts |
| CPU System time | system.cpu.sy.ssh | sy - System calls |
| Kernel thread context switches | system.faults.cs.ssh | cs - Kernel thread context switches |

#### CPU: Breakdown of percentage usage of processor time

| item | key | `vmstat` command parameter |
|-|-|-|
| CPU User time | system.cpu.us.ssh | us - User time. If the current physical processor consumption of the uncapped partitions exceeds the entitled capacity, the percentage becomes relative to the number of physical processor consumed (pc).|
| System calls | system.faults.sy.ssh | sy - System time. If the current physical processor consumption of the uncapped partitions exceeds the entitled capacity, the percentage becomes relative to the number of physical processor consumed (pc). |
| CPU Processor idle time | system.cpu.id.ssh | id - Processor idle time. If the current physical processor consumption of the uncapped partitions exceeds the entitled capacity, the percentage becomes relative to the number of physical processor consumed (pc). |
| CPU Processor iowait time | system.cpu.wa.ssh | wa - Processor idle time during which the system had outstanding disk/NFS I/O request. If the current physical processor consumption of the uncapped partitions exceeds the entitled capacity, the percentage becomes relative to the number of physical processor consumed (pc). |
| CPU Number of physical processors used | system.cpu.pc.ssh | pc - Number of physical processors used. Displayed only if the partition is running with shared processor. |
| CPU Entitled capacity consumed | system.cpu.ec.ssh | ec - The percentage of entitled capacity that is consumed. Displayed only if the partition is running with shared processor. Because the time base over which this data is computed can vary, the entitled capacity percentage can sometimes exceed 100%. This excess is noticeable only with small sampling intervals. |

CPU Additional dependent item:

| item | key |  |
|-|-|-|
| CPU Utilization | system.cpu.util.ssh | CPU utilization in % (100 - CPU idle time)|

Additional info:

source: <https://www.ibm.com/support/knowledgecenter/en/ssw_aix_71/u_commands/uptime.html>

source: <https://www.ibm.com/support/knowledgecenter/en/ssw_aix_71/v_commands/vmstat.html>

source: <https://developer.ibm.com/articles/au-satslowsys/>

## Monitored filesystem parameters

### Discovery rules

| name | key |
|-|-|
| Mounted filesystem discovery | ssh.run[discovery_fs] |

### Filesystem parameters

| item | key |
|-|-|
| Used inodes in number | vfs.fs.inode[{#FSNAME},iused] |
| Used inodes in % | vfs.fs.inode[{#FSNAME},pused] |
| Space utilization % | vfs.fs.size[{#FSNAME},pused] |
| Total space | vfs.fs.size[{#FSNAME},total] |
| Used space | vfs.fs.size[{#FSNAME},used] |
