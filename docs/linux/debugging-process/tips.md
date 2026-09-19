# Linux Debugging Tips

If no processes are listenning on the target port/socket, then check the system logs:

``` bash
journalctl -u nginx
journalctl -p err
cat /var/log/syslog
```

Check disk space: `df -h`

``` bash
$ df -h
Filesystem       Size  Used Avail Use% Mounted on
udev             223M     0  223M   0% /dev
tmpfs             47M  1.6M   45M   4% /run
/dev/nvme1n1p1   7.7G  1.3G  6.0G  18% /
tmpfs            232M     0  232M   0% /dev/shm
tmpfs            5.0M     0  5.0M   0% /run/lock
tmpfs            232M     0  232M   0% /sys/fs/cgroup
/dev/nvme1n1p15  124M  278K  124M   1% /boot/efi
/dev/nvme0n1     8.0G  8.0G   28K 100% /opt/pgdata
tmpfs             47M     0   47M   0% /run/user/1000
```

Change file owners:

``` bash
chown root:www-data <file>
```

!!! note 

    Misconfigured permissions in /var/www/ directly trigger the two most common web server errors:
    ──────
    #### 🛑 Issue 1: 403 Forbidden (Static Files & Directory Traversal)

    To serve a static file (e.g., /var/www/flask_app/static/style.css):

    1. Directory Traversal (+x): Nginx must have Execute (x) permission on every directory in the path (/var,
    /var/www, /var/www/flask_app, and static/) to traverse / enter the folder.
    2. File Read (+r): Nginx must have Read (r) permission on the actual file.

    │ If missing: Nginx cannot open the file and returns HTTP 403 Forbidden (logging 13: Permission denied in
    │ /var/log/nginx/error.log).

     #### 🛑 Issue 2: 502 Bad Gateway (Unix Domain Sockets)

    When proxying traffic to Flask via a Unix socket (e.g., proxy_pass
    http://unix:/var/www/flask_app/flask_app.sock;):

    1. The socket file lives inside /var/www/flask_app/.
    2. The Nginx worker (www-data) must have both Read & Write (rw) permissions on the .sock file to exchange
    HTTP request/response data with Gunicorn or uWSGI.

    │ If missing: Nginx cannot communicate with the socket and returns HTTP 502 Bad Gateway (logging 13:
    │ Permission denied while connecting to upstream in /var/log/nginx/error.log). 