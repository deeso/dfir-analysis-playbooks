## What to do on Linux when triaging and collecting evidence?

Possible project that helps speed up the collection: [Fastir Collector Linux](https://github.com/SekoiaLab/Fastir_Collector_Linux)
Good reading to help understand IR on Linux:
1. [Detecting Linux memfd_create() Fileless Malware with Command Line Forensics](https://web.archive.org/web/20210114181006/https://www.sandflysecurity.com/blog/detecting-linux-memfd_create-fileless-malware-with-command-line-forensics/)
2. [DFIR Blue Team Tips— Finding Evil Process In Linux OS](https://cyberblueteam.medium.com/blue-team-tips-linux-os-finding-evil-running-process-3f12b17c3b8e)
3. [Awesome Incident Response: Linux Evidence Collection](https://github.com/meirwah/awesome-incident-response#linux-evidence-collection)

Understand the context.  Remember to collect the *who*, *what*, *when* and *where*.

1. Collect login information
    1. Determine `who` is logged in
    2. Last logins: `last -Faixw`
    3. Look at auth logs, **Ubuntu:** `cat /var/log/auth.log` and **CentOS:** `cat /var/log/audit/audit.log` **Note: check for rotated logs!**
2. Identify all the users and home directories
    1. Collect all of the `.*_history` files
    2. Collect all the `.known_hosts` files
    3. Check the top most directory `ls -all /` for wierd things like `...` or odd time stamps or permissions
    4. Collect all keys from `.ssh`

4. Look at all the `cron` jobs
    1. `cat /etc/crontab`, review all the potential chron jobs in sub-directories
    2. `crontab -l`
5. Look at all the running services
    1. **all services:** `systemctl list-units --type=service`
    2. **active services:** `systemctl list-units --type=service --state=active`
    3. **running service:** `systemctl --type=service --state=running`

6. interrogate network stacks with `ss`
    1. **summary of socket info :** `ss -s`
    2. **all udp sockets with process extended information (inode, user, etc.):** `ss -eaupo`
    3. **all tcp sockets with process extended information (inode, user, etc.):** `ss -eatpo` **Note: add `-i` if internal tcp state is needed
    5. **all raw sockets with process extended information (inode, user, etc.):** `ss -eawpo`
    6. **all sctp sockets with process extended information (inode, user, etc.):** `ss -eaSpo`
    7. **all dccp sockets with process extended information (inode, user, etc.):** `ss -eadpo`
    8. **all packet sockets with process extended information (inode, user, etc.):** `ss -ea0po`
    9. **all unix sockets with process extended information (inode, user, etc.):** `ss -eaxpo`

7. capture traffic to a file
    1. `tcpdump -i [INTEFACE] -w [FILENAME] -vvvv`
    2. **use extended berkley packet filters (ebpf) for container traffic analysis**

8. Kernel modules loaded, etc.
    1. **list modules:** `lsmod`
    2. **list latent modules on disk:** `for mm in /sys/module/*; do printf "${mm}\n"; done`
    3. **find modules removed from list:** `for mm in /sys/module/*; do if test -d ${mm}/sections; then MOD="$(basename ${mm})"; lsmod | grep -E "^${MOD}" > /dev/null || printf "${MOD}\n"; fi; done` **Note: See [Stackoverflow post](https://stackoverflow.com/questions/13005725/ripping-out-the-hidden-kernel-module-by-reading-kernel-memory-directly)**

9. Process investigation
    1. **process tree:** `ps -auxwf`
    2. **Option 1, enumerate processes in `/proc` with sha256:** `for pid in [0-9]*; do     echo "pid:$pid,cmdline:[$(sudo cat /proc/$pid/cmdline)],fds:$(sudo ls /proc/$pid/fd/ | wc -l),sha256:$(sudo test -f /proc/$pid/exe && sudo sha256sum /proc/$pid/exe | cut -d' ' -f1)"; done` 
    3. **Option 2, enumerate processes in `/proc` without sha256:** `for pid in [0-9]*; do echo "pid:$pid,cmdline:[$(sudo cat /proc/$pid/cmdline)],fds:$(sudo ls /proc/$pid/fd/ | wc -l),ondisk:$(echo $(sudo test -f /proc/$pid/exe && echo 1 || echo 0))"; done`
    4. **Check for deleted files that are still running:**: `ls -alR /proc/*/exe 2> /dev/null | grep deleted`
    5. **Understand file descriptors belong to a PID**: ``
    6. **Review the environmental variables**: `sudo strings /proc/$PID_OF_INTEREST/envion`
    7. **Dump processes with [AVML](https://github.com/microsoft/avml). If the process is large create a RAMDISK!!!!, see: [Creating a ram disk on Linux](https://unix.stackexchange.com/questions/66329/creating-a-ram-disk-on-linux)**
        1. **Create a mount point for the data for our process (Replace REPLACEME_ with the ACTUAL size you want):** `sudo mkdir /mnt/dumped_process && sudo mount -o size=REPLACEME_G -t tmpfs none /mnt/tmpfs`
    8. **Leverage Volatility to perform the memory analysis, where possible**

10. Capture all the server and application logs in `/var/log`
11. Enumerate files that have been out or that are unexpected with `find`
12. Handling Docker Containers (**see [Container Forensics with Docker Explorer](https://osdfir.blogspot.com/2021/01/container-forensics-with-docker-explorer.html) for a good case study**)
    1. **obtain an acquisition of where the containers ran (or do it on the host machine)**
    2. **download and install docker explorer**: [Docker Explorer](https://github.com/google/docker-explorer)
    3. **identify container of interest**
    4. **analyze container logs**
    5. **mount and analyze the container filesystem**
    6. **review the container history**

