# Systemd Basics

**What is systemd?**

- It's an **init system**.
- The init system is the most important process running on your server (PID 1).
- It manages all services that run in the background.

Systemd is the most common init system in Linux. The init system is the very first process that starts on your system and its job is to schedule all other processes on that system. 

**Working with units**

- Units in systemd are resources that it's able to manage.
- These include services, timers, mounts, automounts, and more.
- Service is a type of unit.

--- 

## Basic commands

Install apache: 

``` bash
$ sudo apt install apache2
```

Check the apache service status: `sudo systemctl status apache2`

``` bash
$ sudo systemctl status apache2
● apache2.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/apache2.service; enabled; preset: enabled)
     Active: active (running) since Thu 2026-09-10 01:15:32 EDT; 18s ago
 Invocation: ab39578c43ac4325a03849b901fc66b0
       Docs: https://httpd.apache.org/docs/2.4/
   Main PID: 10134 (apache2)
     Status: "Total requests: 0; Idle/Busy workers 100/0;Requests/sec: 0; Bytes served/sec:   0 B/sec"
      Tasks: 55 (limit: 15099)
     Memory: 5.7M (peak: 6.2M)
        CPU: 94ms
     CGroup: /system.slice/apache2.service
             ├─10134 /usr/sbin/apache2 -k start -DFOREGROUND
             ├─10150 /usr/sbin/apache2 -k start -DFOREGROUND
             └─10151 /usr/sbin/apache2 -k start -DFOREGROUND

Sep 10 01:15:32 clicknam-HP-EliteDesk-800-G3-DM-35W systemd[1]: Starting apache2.service - The Apache HTTP Server...
Sep 10 01:15:32 clicknam-HP-EliteDesk-800-G3-DM-35W apachectl[10134]: AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.1>
```
- Start the apache: `sudo systemctl start apache2`
- Stop the service: `sudo systemctl stop apache2`
- Restart the service(used when the configuration is changed): `sudo systemctl restart apache2`


Disable the apache2:

``` bash
$ sudo systemctl disable apache2
[sudo: authenticate] Password:
Synchronizing state of apache2.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install disable apache2
Removed '/etc/systemd/system/multi-user.target.wants/apache2.service'.

$ sudo systemctl status apache2
● apache2.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/apache2.service; disabled; preset: enabled)
     Active: active (running) since Thu 2026-09-10 01:15:32 EDT; 24min ago
 Invocation: ab39578c43ac4325a03849b901fc66b0
       Docs: https://httpd.apache.org/docs/2.4/
   Main PID: 10134 (apache2)
     Status: "Total requests: 6; Idle/Busy workers 100/0;Requests/sec: 0.00403; Bytes served/sec:  14 B/sec"
      Tasks: 55 (limit: 15099)
     Memory: 6.4M (peak: 6.7M)
        CPU: 225ms
     CGroup: /system.slice/apache2.service
             ├─10134 /usr/sbin/apache2 -k start -DFOREGROUND
             ├─10150 /usr/sbin/apache2 -k start -DFOREGROUND
             └─10151 /usr/sbin/apache2 -k start -DFOREGROUND

Sep 10 01:15:32 clicknam-HP-EliteDesk-800-G3-DM-35W systemd[1]: Starting apache2.service - The Apache HTTP Server...
Sep 10 01:15:32 clicknam-HP-EliteDesk-800-G3-DM-35W apachectl[10134]: AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.1>
```

Enable it again:

``` bash
clicknam@clicknam-HP-EliteDesk-800-G3-DM-35W:~$ sudo systemctl enable apache2
Synchronizing state of apache2.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install enable apache2
Created symlink '/etc/systemd/system/multi-user.target.wants/apache2.service' → '/usr/lib/systemd/system/apache2.service'.
```

---

### Systemd unit directory priority

1. `/etc/systemd/system`: the highest priority
1. `/run/systemd/system`
1. `/lib/systemd/system`

When the linux system is started, it'll start systemd and systemd will go into these directories looking for unit files. If it finds any then it's going to load those into memory.

---

## Service files

Service files are text files that contain instructions that tell systemd how it needs to manage that particular service. 

``` bash title="systemctl cat apache2"
$ cat /usr/lib/systemd/system/apache2.service
[Unit]
Description=The Apache HTTP Server
After=network.target remote-fs.target nss-lookup.target
Documentation=https://httpd.apache.org/docs/2.4/

[Service]
Type=notify
Environment=APACHE_STARTED_BY_SYSTEMD=true
ExecStart=/usr/sbin/apachectl start
ExecStop=/usr/sbin/apachectl graceful-stop
ExecReload=/usr/sbin/apachectl graceful
# Send SIGWINCH for graceful stop
KillSignal=SIGWINCH
KillMode=mixed
PrivateTmp=true
Restart=on-abnormal
OOMPolicy=continue
RemoveIPC=yes

...

[Install]
WantedBy=multi-user.target
```

### Sections

- `Unit`: contains general info about the unit.
    - `After=network.target remote-fs.target nss-lookup.target`: `After` outlines that something else is required and it must happen before this unit file can start up. 
        - `network.target` represents `/usr/lib/systemd/system/network.target`
        - `remote-fs.target` represents `/usr/lib/systemd/system/remote-fs.target`
        - `nss-lookup.target` represents `/usr/lib/systemd/system/nss-lookup.target`
- `Service`: configuration options
    - `Type`: 
        - `simple`(the default type) causes systemd to consider the service to be started up as soon as you start it.
        - `notify` is the same except it's not configured to be running until the process tells systemd that it's ready.
        - `forking` means the process is considered to be started when other processes are spawned from a parent process and then that parent process stops to where the children are the only processes that are running. 
    - `ExecStart`: the actual command for `sudo systemctl start apache2`
    - `ExecStop`: the actual command for `sudo systemctl stop apache2`
    - `ExecReload`: the actual command for `sudo systemctl reload apache2`
- `Install`: configure what happens when a unit file is enabled or disabled.
    - A unit that's enabled will start up when the server starts up, and disabled is the exact opposite of that. When a service is enabled we can use this section to set up how that's handled.
    - `WantedBy`: `multi-user.target` defines that the system must have reached a state where multiple users can use the system at the same time so in this case Apache will not start until that stage has been reached.

### Edit the unit file

#### sudo systemctl edit apache2.service

`sudo systemctl edit apache2.service` creates an config override file at `/etc/systemd/system/apache2.service.d/override.conf`.

``` bash hl_lines="2 7-8"
$ sudo systemctl edit apache2.service
### Editing /etc/systemd/system/apache2.service.d/override.conf
### Anything between here and the comment below will become the contents of the drop-in file

### Edits below this comment will be discarded

[Unit]
Description=The Apache HTTP Server(this is override description)

### /usr/lib/systemd/system/apache2.service
# [Unit]
# Description=The Apache HTTP Server(this is override description)
# After=network.target remote-fs.target nss-lookup.target
# Documentation=https://httpd.apache.org/docs/2.4/
#
# [Service]
# Type=notify
# Environment=APACHE_STARTED_BY_SYSTEMD=true
```

- We are currently editing `/etc/systemd/system/apache2.service.d/override.conf` file which have a priority over the default configs in `/usr/lib/systemd/system/apache2.service`.
- Overrride `Description` config.

#### sudo systemctl edit --full apache2.service

Adding `--full` option will directly create a new unit file in `/etc/systemd/system` that priortizes the unit file in `/usr/lib/systemd/system/apache2.service`.

``` bash hl_lines="4"
$ sudo systemctl edit --full apache2.service
 GNU nano 8.7.1                                               /etc/systemd/system/.#apache2.serviceb7ddc1e8dadace96
[Unit]
Description=This is ovrride apache2 description
After=network.target remote-fs.target nss-lookup.target
Documentation=https://httpd.apache.org/docs/2.4/

[Service]
Type=notify
Environment=APACHE_STARTED_BY_SYSTEMD=true
ExecStart=/usr/sbin/apachectl start
ExecStop=/usr/sbin/apachectl graceful-stop
ExecReload=/usr/sbin/apachectl graceful
# Send SIGWINCH for graceful stop
KillSignal=SIGWINCH
KillMode=mixed
```


``` bash
$ diff /etc/systemd/system/apache2.service /usr/lib/systemd/system/apache2.service
2c2
< Description=This is ovrride apache2 description
---
> Description=The Apache HTTP Server
```

Reload the configs and check the apache2 status. You will find the description has been changed.

``` bash hl_lines="1-3"
$ sudo systemctl daemon-reload
$ sudo systemctl status apache2
● apache2.service - This is ovrride apache2 description
     Loaded: loaded (/etc/systemd/system/apache2.service; enabled; preset: enabled)
     Active: active (running) since Thu 2026-09-10 01:15:32 EDT; 10h ago
 Invocation: ab39578c43ac4325a03849b901fc66b0
       Docs: https://httpd.apache.org/docs/2.4/
   Main PID: 10134 (apache2)
     Status: "Total requests: 8; Idle/Busy workers 100/0;Requests/sec: 0.000206; Bytes served/sec:   0 B/sec"
      Tasks: 55 (limit: 15099)
     Memory: 6.9M (peak: 7.2M)
        CPU: 3.491s
     CGroup: /system.slice/apache2.service
             ├─10134 /usr/sbin/apache2 -k start -DFOREGROUND
             ├─10150 /usr/sbin/apache2 -k start -DFOREGROUND
             └─10151 /usr/sbin/apache2 -k start -DFOREGROUND

Sep 10 01:15:32 clicknam-HP-EliteDesk-800-G3-DM-35W systemd[1]: Starting apache2.service - The Apache HTTP Server...
Sep 10 01:15:32 clicknam-HP-EliteDesk-800-G3-DM-35W apachectl[10134]: AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.1>
Sep 10 01:15:32 clicknam-HP-EliteDesk-800-G3-DM-35W systemd[1]: Started apache2.service - The Apache HTTP Server.
```

---

## Reference

<iframe width="560" height="315" src="https://www.youtube.com/embed/Kzpm-rGAXos?si=UOY74BEJkQzehfL0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>