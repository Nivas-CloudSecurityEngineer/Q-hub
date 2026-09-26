<div align="center" markdown="1">

# 🐧 Linux (DevOps Focused)
### 🎤 Interview Questions & Answers

![Category](https://img.shields.io/badge/Category-Linux-blue?style=for-the-badge&logo=linux&logoColor=white)
![Level](https://img.shields.io/badge/Level-Beginner_to_Advanced-yellow?style=for-the-badge)
![Focus](https://img.shields.io/badge/Focus-Real--Time%20Scenarios-success?style=for-the-badge)

</div>

---

> 💡 **Pro Tip:** Skim the 🧩 fundamentals first, then master the 🎯 real-time scenarios — that's where interviews are actually won or lost.

---

## 📁 Basics & File System

<details markdown="1">
<summary>❓ <b>1. What is Linux and why is it preferred for DevOps/servers?</b></summary>
<br>

Linux is an open-source, Unix-like OS kernel. It's preferred for servers because it's stable, secure, lightweight, free, highly customizable, and has strong CLI/scripting support that suits automation-heavy DevOps workflows.

</details>

<details markdown="1">
<summary>❓ <b>2. Explain the Linux file system hierarchy.</b></summary>
<br>

- `/` - root of the filesystem
- `/etc` - configuration files
- `/var` - variable data (logs, spool, cache) - `/var/log` is critical for troubleshooting
- `/home` - user home directories
- `/bin`, `/sbin` - essential binaries
- `/usr` - user programs/libraries
- `/opt` - optional/third-party software
- `/tmp` - temporary files
- `/proc`, `/sys` - virtual filesystems exposing kernel/process info

</details>

<details markdown="1">
<summary>❓ <b>3. Difference between hard link and soft link (symlink)?</b></summary>
<br>

A hard link points directly to the same inode as the original file, shares the same data blocks, and survives deletion of the original. A soft link is a pointer/shortcut to a path; it breaks if the original is deleted and can cross filesystems.

</details>

<details markdown="1">
<summary>❓ <b>4. How do you find which process is using a specific port?</b></summary>
<br>

`sudo lsof -i :8080` or `sudo netstat -tulnp | grep 8080` or `sudo ss -tulnp | grep 8080`.

</details>

<details markdown="1">
<summary>❓ <b>5. How do you check disk usage and free space?</b></summary>
<br>

`df -h` for filesystem-level usage, `du -sh /path/*` for directory/file-level usage.

</details>

<details markdown="1">
<summary>🎯 <b>6. Scenario: A server's `/` partition is 100% full. How do you troubleshoot?</b></summary>
<br>

Run `df -h` to confirm, then `du -sh /* | sort -rh` to find big directories (commonly `/var/log`, `/tmp`, docker images in `/var/lib/docker`). Clear rotated/old logs, truncate large open log files (`> file.log`), clear package caches (`apt clean`/`yum clean all`), remove unused Docker images (`docker system prune`), and check for deleted-but-open file handles with `lsof +L1`.

</details>

<details markdown="1">
<summary>❓ <b>7. What are inodes? What happens if inodes are exhausted even with free disk space?</b></summary>
<br>

An inode stores metadata about a file (permissions, owner, size, pointers to data blocks) but not the filename. If inodes run out (common with millions of small files, e.g., session/cache files), you can't create new files even if disk space is available. Check with `df -i`.

</details>

## 👤 Permissions & Users

<details markdown="1">
<summary>❓ <b>8. Explain Linux file permissions (rwx) and how to read `-rwxr-xr--`.</b></summary>
<br>

r=read, w=write, x=execute, for three classes: owner, group, others. `-rwxr-xr--` means: it's a file, owner has rwx, group has r-x, others have r-- only.

</details>

<details markdown="1">
<summary>❓ <b>9. Difference between `chmod 755` and `chmod 644`?</b></summary>
<br>

755 = rwxr-xr-x (owner full, group/others read+execute) - typical for scripts/executables. 644 = rw-r--r-- (owner read/write, others read-only) - typical for regular config/data files.

</details>

<details markdown="1">
<summary>❓ <b>10. What is umask and how does it affect new files?</b></summary>
<br>

umask defines default permission mask subtracted from base permissions (666 for files, 777 for dirs) when new files/dirs are created. E.g., umask 022 gives files 644 and dirs 755.

</details>

<details markdown="1">
<summary>❓ <b>11. Difference between SUID, SGID, and Sticky bit.</b></summary>
<br>

- SUID: file executes with owner's privileges (e.g., `passwd`).
- SGID: file/dir executes with group privileges; on a directory, new files inherit the group.
- Sticky bit: on a directory (like `/tmp`), only file owner/root can delete/rename files even if others have write access.

</details>

<details markdown="1">
<summary>❓ <b>12. How do you add a user, add to a group, and grant sudo access?</b></summary>
<br>

`sudo useradd -m -s /bin/bash username`, set password with `sudo passwd username`, add to group via `sudo usermod -aG sudo username` (Debian/Ubuntu) or `wheel` group (RHEL/CentOS).

</details>

<details markdown="1">
<summary>🎯 <b>13. Scenario: A deployment script fails with "Permission Denied" on a shell script. How do you fix it?</b></summary>
<br>

Check execute bit with `ls -l script.sh`; add with `chmod +x script.sh`. Also verify the shebang line (`#!/bin/bash`) and that the filesystem isn't mounted with `noexec`.

</details>

## ⚙️ Process Management

<details markdown="1">
<summary>❓ <b>14. How do you list running processes and kill one?</b></summary>
<br>

`ps aux` or `top`/`htop` to list; `kill -9 <PID>` for force kill, `kill -15 <PID>` for graceful termination (SIGTERM).

</details>

<details markdown="1">
<summary>❓ <b>15. Difference between `kill`, `kill -9`, and `killall`.</b></summary>
<br>

`kill` sends SIGTERM (15) by default, allowing graceful shutdown. `kill -9` sends SIGKILL, an immediate forced termination the process can't catch/ignore. `killall` kills all processes matching a name.

</details>

<details markdown="1">
<summary>🎯 <b>16. Scenario: An application is consuming 100% CPU. How do you troubleshoot?</b></summary>
<br>

Use `top`/`htop` to identify the PID, `ps -p <PID> -o %cpu,%mem,cmd`. Use `strace -p <PID>` to see syscalls, `jstack`/`pidstat` for thread-level analysis (Java apps), check logs, and consider `nice`/`renice` to deprioritize while investigating, or restart the service if it's a known memory/CPU leak.

</details>

<details markdown="1">
<summary>❓ <b>17. What is a zombie process and how do you deal with it?</b></summary>
<br>

A zombie is a terminated process whose exit status hasn't been reaped by its parent; it shows as `Z` in `ps`. It consumes no resources except a PID entry. Fix by getting the parent process to call `wait()`, or by killing/restarting the parent if it's misbehaving.

</details>

<details markdown="1">
<summary>❓ <b>18. Difference between a process and a thread.</b></summary>
<br>

A process has its own memory space and resources; a thread is a lightweight unit of execution within a process, sharing memory with other threads of the same process.

</details>

<details markdown="1">
<summary>❓ <b>19. How do you run a process in the background and keep it running after logout?</b></summary>
<br>

Use `&` to background it, `nohup command &` or `disown` to detach from the terminal, or better, use `screen`/`tmux`, or run it as a `systemd` service for production use.

</details>

## 🌐 Networking

<details markdown="1">
<summary>❓ <b>20. How do you check network connectivity and DNS resolution?</b></summary>
<br>

`ping host`, `traceroute host`, `curl -v url`, `nslookup domain` / `dig domain` for DNS, `telnet host port` or `nc -zv host port` for port connectivity.

</details>

<details markdown="1">
<summary>🎯 <b>21. Scenario: Application can't connect to a database on another server. How do you troubleshoot?</b></summary>
<br>

1) Check DNS resolves: `nslookup db-host`. 2) Check network reachability: `ping`, `telnet db-host 3306`/`nc -zv`. 3) Check firewall rules: `iptables -L` / security groups. 4) Check the DB service is listening: `netstat -tulnp | grep 3306` on the DB server. 5) Check app logs/credentials/connection string.

</details>

<details markdown="1">
<summary>❓ <b>22. Explain the difference between TCP and UDP.</b></summary>
<br>

TCP is connection-oriented, reliable, ordered (handshake, retransmission) - used for HTTP, SSH, DB connections. UDP is connectionless, faster, no delivery guarantee - used for DNS, streaming, VoIP.

</details>

<details markdown="1">
<summary>❓ <b>23. What is the purpose of `/etc/hosts` and `/etc/resolv.conf`?</b></summary>
<br>

`/etc/hosts` maps hostnames to IPs locally (checked before DNS). `/etc/resolv.conf` configures DNS resolvers the system queries.

</details>

<details markdown="1">
<summary>❓ <b>24. How do you configure a static IP on Linux?</b></summary>
<br>

Depends on distro: via Netplan (Ubuntu, YAML in `/etc/netplan/`), `nmcli`/`nmtui` (NetworkManager), or editing `/etc/network/interfaces` (Debian) / ifcfg files (RHEL), then applying/restarting networking.

</details>

## 🧾 Logging & Monitoring

<details markdown="1">
<summary>❓ <b>25. Where are system logs stored and how do you view them?</b></summary>
<br>

`/var/log/` (e.g., `syslog`, `messages`, `auth.log`, `secure`). Use `tail -f /var/log/syslog` for live tail, or `journalctl` on systemd-based systems.

</details>

<details markdown="1">
<summary>❓ <b>26. How do you use `journalctl` to debug a failed service?</b></summary>
<br>

`journalctl -u servicename` for logs of a specific unit, `journalctl -u servicename -f` to follow live, `journalctl -p err` to filter errors, `journalctl --since "1 hour ago"`.

</details>

<details markdown="1">
<summary>🎯 <b>27. Scenario: An SSH login is failing intermittently for a user. Where do you check?</b></summary>
<br>

`/var/log/auth.log` (Debian/Ubuntu) or `/var/log/secure` (RHEL) for authentication attempts and errors; check `sshd_config`, key permissions (`~/.ssh` must be 700, `authorized_keys` 600), and account lock status (`faillock`/`pam_tally2`).

</details>

## 🛎️ systemd & Services

<details markdown="1">
<summary>❓ <b>28. How do you create and manage a custom systemd service?</b></summary>
<br>

Create a unit file in `/etc/systemd/system/myapp.service` with `[Unit]`, `[Service]`, `[Install]` sections, then `systemctl daemon-reload`, `systemctl enable --now myapp`.

</details>

<details markdown="1">
<summary>❓ <b>29. Difference between `systemctl restart` and `systemctl reload`.</b></summary>
<br>

`restart` stops and starts the whole service (brief downtime). `reload` tells the service to re-read its configuration without stopping the process (if supported), avoiding downtime.

</details>

<details markdown="1">
<summary>🎯 <b>30. Scenario: A service keeps crashing/restarting in a loop. How do you debug and prevent flapping?</b></summary>
<br>

Check `systemctl status service` and `journalctl -u service` for the crash reason. Use `Restart=on-failure` with `RestartSec` and `StartLimitBurst`/`StartLimitIntervalSec` in the unit file to control restart behavior and avoid a crash loop hammering the system.

</details>

## 📝 Shell Scripting & Automation

<details markdown="1">
<summary>❓ <b>31. Write a basic script to check if a service is running and restart it if not.</b></summary>
<br>

```bash
#!/bin/bash
if ! systemctl is-active --quiet nginx; then
  systemctl restart nginx
  echo "$(date): nginx restarted" >> /var/log/nginx_monitor.log
fi
```

</details>

<details markdown="1">
<summary>❓ <b>32. Difference between `$*` and `$@` in shell scripts.</b></summary>
<br>

`$*` treats all arguments as a single string; `$@` treats each argument as a separate quoted string when used as `"$@"` - important in loops to preserve arguments with spaces.

</details>

<details markdown="1">
<summary>❓ <b>33. How do you schedule recurring tasks in Linux?</b></summary>
<br>

Using `cron` (edit with `crontab -e`) for user/scheduled jobs, or `systemd timers` for more robust logging/dependency management.

</details>

<details markdown="1">
<summary>🎯 <b>34. Scenario: A cron job isn't running as expected, though it works manually. Why?</b></summary>
<br>

Common causes: cron uses a minimal environment (missing `PATH`/env vars), relative paths used instead of absolute, script isn't executable, output/errors not redirected/logged (add `>> /var/log/job.log 2>&1`), or wrong user's crontab.

</details>

<details markdown="1">
<summary>❓ <b>35. How do you find and delete files older than 7 days?</b></summary>
<br>

`find /path -type f -mtime +7 -exec rm -f {} \;` (add `-name "*.log"` to scope it, and always test with `-print` first before deleting).

</details>

## 🎯 Real-Time Scenarios

<details markdown="1">
<summary>🎯 <b>36. Scenario: Load average is high but CPU usage looks low. What could be happening?</b></summary>
<br>

High load average with low CPU usage often indicates processes stuck in uninterruptible sleep (`D` state), usually waiting on disk I/O. Check with `ps aux | awk '$8=="D"'` and `iostat`/`iotop` to find the disk bottleneck.

</details>

<details markdown="1">
<summary>🎯 <b>37. Scenario: You need to securely copy a large file to 50 servers. What's your approach?</b></summary>
<br>

Use `rsync` (efficient, resumable, delta transfer) over SSH, or better, use configuration management (Ansible) with a `copy`/`fetch` module, or push it to S3/artifact repo and pull from each server in parallel via automation, avoiding sequential scp loops.

</details>

<details markdown="1">
<summary>🎯 <b>38. Scenario: A production server's time is out of sync causing failed TLS/cert validation. How do you fix and prevent it?</b></summary>
<br>

Fix immediately with `chronyc makestep` or `ntpdate`; ensure `chronyd`/`ntpd` service is enabled and running for continuous sync; verify with `timedatectl` and `chronyc tracking`.

</details>

<details markdown="1">
<summary>❓ <b>39. How do you troubleshoot high memory usage on a Linux server?</b></summary>
<br>

`free -h` for overview, `top`/`htop` sorted by memory, check for OOM killer activity in `dmesg`/`journalctl -k | grep -i "out of memory"`, review swap usage, and identify memory leaks in application logs.

</details>

<details markdown="1">
<summary>❓ <b>40. Difference between swap and RAM, and when is high swap usage a red flag?</b></summary>
<br>

RAM is fast volatile memory; swap is disk space used as overflow when RAM is full. High swap usage is a red flag because disk is orders of magnitude slower - it indicates memory pressure and can severely degrade performance; the fix is adding RAM, tuning `vm.swappiness`, or fixing memory-leaking apps.

</details>

<details markdown="1">
<summary>❓ <b>41. How would you harden a Linux server for production/security?</b></summary>
<br>

Disable root SSH login, use key-based auth, enable a firewall (ufw/iptables/firewalld), keep packages patched, disable unused services, enforce least-privilege sudo, enable fail2ban, configure SELinux/AppArmor, and enable centralized logging/auditing (auditd).

</details>

<details markdown="1">
<summary>🎯 <b>42. Scenario: You need to identify what changed on a server before an outage. What do you check?</b></summary>
<br>

Check `journalctl`/`/var/log` around the incident time, package manager history (`yum history`/`apt history.log`), config management run logs (Ansible/Puppet/Chef), `auditd` logs, and shell history (`~/.bash_history`) if applicable, correlating with monitoring/alerting timestamps.

</details>

<details markdown="1">
<summary>❓ <b>43. What's the difference between `yum`/`apt` and how do you manage packages?</b></summary>
<br>

`apt` (Debian/Ubuntu) and `yum`/`dnf` (RHEL/CentOS/Fedora) are package managers handling install/upgrade/remove and dependency resolution from repositories, e.g., `apt install nginx`, `yum install httpd`.

</details>

<details markdown="1">
<summary>❓ <b>44. How do you check open file limits and increase them for a high-throughput application?</b></summary>
<br>

Check with `ulimit -n`; view/modify persistent limits in `/etc/security/limits.conf` and systemd unit `LimitNOFILE=` for services, since the default (1024) is often too low for busy web/DB servers.

</details>

<details markdown="1">
<summary>🎯 <b>45. Scenario: Users report a website is slow. As a Linux admin, what's your triage order?</b></summary>
<br>

1) Check server resources (`top`, `free`, `df`, `iostat`). 2) Check network latency (`ping`, `traceroute`). 3) Check app/web server logs for errors/slow queries. 4) Check load balancer/health checks. 5) Check DB performance. 6) Correlate with monitoring dashboards (Grafana/CloudWatch) for the timeframe of the complaint.

</details>

