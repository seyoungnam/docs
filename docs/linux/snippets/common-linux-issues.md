# 10 Common Linux Issues and How to Fix them



## SSH Failures

`ssh <IP>` command returns an error with the "Connection refused" message.

``` bash
$ ssh 192.168.8.104
ssh: connect to host 192.168.8.104 port 22: Connection refused
```

Check if the SSH daemon(`sshd`) is running on the linux server:

``` bash title="sudo systemctl status ssh"
$ sudo systemctl status ssh
[sudo: authenticate] Password:
Unit ssh.service could not be found.
```

If not installed or inactive, install and start it:

``` bash
sudo apt update && sudo apt install openssh-server -y
sudo systemctl enable --now ssh
```

Confirm the SSH port and listeneing status

``` bash
$ sudo ss -tulpn | grep -i ssh
tcp   LISTEN 0      4096         0.0.0.0:22        0.0.0.0:*    users:(("sshd",pid=42069,fd=3),("systemd",pid=1,fd=139))
tcp   LISTEN 0      4096            [::]:22           [::]:*    users:(("sshd",pid=42069,fd=4),("systemd",pid=1,fd=143))

$ sudo lsof -i :22
COMMAND   PID USER  FD   TYPE DEVICE SIZE/OFF NODE NAME
systemd     1 root 139u  IPv4 122086      0t0  TCP *:ssh (LISTEN)
systemd     1 root 143u  IPv6 124308      0t0  TCP *:ssh (LISTEN)
sshd    42069 root   3u  IPv4 122086      0t0  TCP *:ssh (LISTEN)
sshd    42069 root   4u  IPv6 124308      0t0  TCP *:ssh (LISTEN)
```

- a process named `sshd` in the `LISTEN` state bound to `0.0.0.0:22` or `[::]:22`

Check local firewall rules

``` bash 
$ sudo ufw status
Status: inactive

$ sudo ufw allow ssh
Rules updated
Rules updated (v6)

$ sudo ufw reload
Firewall not enabled (skipping reload)

$ sudo ufw enable
Firewall is active and enabled on system startup

$ sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
22/tcp (v6)                ALLOW       Anywhere (v6)
```

Test the connection locally in your linux server

``` bash
ssh -v <your-username>@127.0.0.1
```

Try to connect from the client side:

``` bash hl_lines="6"
# test
nc -vz 192.168.8.104 22
Connection to 192.168.8.104 port 22 [tcp/ssh] succeeded!

# 
$ ssh <your-username>@192.168.8.104
<your-username>@192.168.8.104's password:
Welcome to Ubuntu 26.04 LTS (GNU/Linux 7.0.0-30-generic x86_64)

...
Last login: Thu Sep 10 14:15:23 2026 from 127.0.0.1
```

On the server side, you will see the log associated with this connection by `tail -f /var/log/auth.log`:

``` bash hl_lines="4-6"
$ tail -n 5 /var/log/auth.log
2026-09-10T14:21:29.874066-04:00 clicknam-HP-EliteDesk-800-G3-DM-35W sshd-session[42995]: Connection closed by invalid user ec2-user 192.168.8.107 port 60491 [preauth]
2026-09-10T14:21:29.875150-04:00 clicknam-HP-EliteDesk-800-G3-DM-35W sshd-session[42995]: PAM 1 more authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=192.168.8.107
2026-09-10T14:21:46.649859-04:00 clicknam-HP-EliteDesk-800-G3-DM-35W sshd-session[43004]: Accepted password for clicknam from 192.168.8.107 port 60499 ssh2
2026-09-10T14:21:46.651170-04:00 clicknam-HP-EliteDesk-800-G3-DM-35W sshd-session[43004]: pam_unix(sshd:session): session opened for user clicknam(uid=1000) by clicknam(uid=0)
2026-09-10T14:21:46.655564-04:00 clicknam-HP-EliteDesk-800-G3-DM-35W systemd-logind[5119]: New session '29' of user 'clicknam' with class 'user' and type 'tty'
```

---

## Strange disk full errors

You can't save a file with the `No space left on device` error message. But when you check the disk isn't full.

``` bash title="df -h"
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        50G   20G   28G  42% /
/dev/sdb1       500G  450G   25G  95% /data
```

Try `df -i` instead. It shows you how many inodes you have free. Inodes represent the number of filesystem entries(files, folders, symlinks). A linux installation has a limited number of inodes. Every file or object is going to take up one of these inodes. That means you have a finite number of files you could store on a linux system. 

``` bash title="df -i" hl_lines="4"
$ df -i
Filesystem       Inodes  IUsed   IFree IUse% Mounted on
/dev/sda1       3276800 120400 3156400    4% /
/dev/sdb1      32768000 32768000     0  100% /data
```
- The output of `df -i` shows that `/dev/sdb1` has 0 free inodes(`IFree = 0`, `IUse% = 100%`), meaning it cannot create any new files even if physical storage space is still available.


---

## Reference

<iframe width="560" height="315" src="https://www.youtube.com/embed/xsdFNpThetE?si=sNX0xAfZb10V8GAU" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>