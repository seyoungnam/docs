# Web Server Debugging Process

**Request Flow**

```
[Client / Route 53] ──> [EC2 / Nginx :80] ──(Unix Socket)──> [uWSGI / Flask] ──> [SQLite DB]
```

**Typical Bug Flow Across the Stack:**

1. Network & Security Group Layer
    - AWS Security Group or local firewall (`iptables` / `ufw`) blocking incoming TCP port 80/443 traffic.
    - Route 53 or /etc/hosts pointing to an incorrect IP.

1. NginX Layer
    - Service inactive or failing to start (`systemctl status nginx`).
    - Configuration syntax error in `/etc/nginx/sites-available/` or `/etc/nginx/nginx.conf` (`nginx -t`).
    - Incorrect `proxy_pass` URI or missing upstream definition.
    - Permission denied (`403` / `502`) accessing the Unix socket file.

1. Unix Domain Socket & uWSGI Layer
    - Socket file path mismatch between Nginx config and uWSGI config (e.g., `/tmp/app.sock` vs `/run/uwsgi/app.sock`).
    - File ownership/permission mismatch (Nginx running as `www-data` cannot read/write the socket owned by a different user).
    - uWSGI failing to load the Flask entry point (e.g., misconfigured `module = wsgi:app` or missing virtualenv path).

1. Flask Application Layer
    - Python runtime syntax errors or unhandled exceptions in the route handlers.
    - Missing Python environment variables or uninstalled pip packages.
    - Logic bugs in request parameter parsing or JSON serialization.

1. SQLite Layter
    - File permission issues on the `.db` file or parent directory (the application user has read permissions but lacks write permissions for WAL/journaling).
    - Unapplied database migrations (missing tables or missing columns).
    - Malformed SQL queries (e.g., syntax errors, unescaped queries, or constraint violations).

---
## 1. Verify the Edge and Ingress

### Test external connectivity / HTTP response

``` bash
curl -v http://localhost
curl -v http://127.0.0.1:80
```

### Check listening ports

``` bash title="sudo ss -tulpn" hl_lines="2"
$ sudo ss -tulpn | grep -E ':(http|80|443|5000)'
tcp   LISTEN 0      511           0.0.0.0:80         0.0.0.0:*    users:(("nginx",pid=771,fd=5),("nginx",pid=769,fd=5),("nginx",pid=767,fd=5))
tcp   LISTEN 0      4096                *:8080             *:*    users:(("gotty",pid=698,fd=6))
```

- `LISTEN`: Nginx has opened the socket and is actively waiting for incoming HTTP connections.
- `0.0.0.0:80`: Nginx is bound to Port 80 on all IPv4 network interfaces (accessible from external network, not just localhost).
- `users:(("nginx",pid=771,fd=5),...`: The listening socket is **File Descriptor** `fd=5`, shared by the Master process (`PID 767`) and its Worker processes (`PID 769` and `PID 771`).
- `gotty` (`PID 698`): A web-based terminal utility is running and listening on Port 8080 (`*:8080` on both IPv4 and IPv6).

### Review system logs

If no processes are listenning on the target port/socket, then check the system logs:

``` bash
journalctl -u nginx
journalctl -p err
cat /var/log/syslog
```



---

## 2. Inspect Nginx

The main troubleshooting path is:

1. Check process lifecycle & PIDs (`systemctl status nginx`)
1. Locate logs & global settings (`/etc/nginx/nginx.conf`)
1. Inspect the actual reverse proxy routing (`/etc/nginx/sites-enabled/flask_app`)
1. Check Socket File Permission(`ls -l /var/www/flask_app/flask_app.sock`)
1. Test Nginx config after any config changes (`sudo nginx -t`)

### Check process lifecycle

``` bash hl_lines="3 4 12-14" title="systemctl status nginx"
$ systemctl status nginx
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: enabled) # (1)!
     Active: active (running) since Sun 2026-09-06 03:22:54 UTC; 1min 45s ago # (2)!
 Invocation: 5e4310b2e74a4478b632ec3d95bf63fa
       Docs: man:nginx(8)
   Main PID: 783 (nginx)
      Tasks: 3 (limit: 501)
     Memory: 4.2M (peak: 4.3M)
        CPU: 42ms
     CGroup: /system.slice/nginx.service
             ├─783 "nginx: master process /usr/sbin/nginx -g daemon on; master_process on;" # (3)!
             ├─784 "nginx: worker process" # (4)!
             └─785 "nginx: worker process"

Sep 06 03:22:54 i-0b1eba0e1766d6850 systemd[1]: Starting nginx.service - A high performance web server and a reverse proxy server...
Sep 06 03:22:54 i-0b1eba0e1766d6850 systemd[1]: Started nginx.service - A high performance web server and a reverse proxy server.
```

1.  `/usr/lib/systemd/system/nginx.service` is the nginx main config file read by Linux(`systemd`) to manage the OS process lifecycle. `systemctl cat nginx` prints out the file content.
2.  Ensure the `active` status.
3.  `783` is the nginx master process ID.
4.  `784` and `785` are the worker process ID.

!!! note "How Nginx gets started"

    When you start Nginx, the two files(nginx.service and nginx.conf) work in sequence:

    1. You run: `sudo systemctl start nginx`
    2. systemd reads: `/usr/lib/systemd/system/nginx.service`
    3. systemd executes: `/usr/sbin/nginx` (the binary)
    4. Nginx binary reads: `/etc/nginx/nginx.conf`
    5. Nginx binds to ports 80/443 and begins accepting HTTP requests

### Check Nginx global settings

``` bash title="/etc/nginx/nginx.conf" hl_lines="6 7 21 27 32-33"
$ cat /etc/nginx/nginx.conf
user www-data;
worker_processes auto;
worker_cpu_affinity auto;
pid /run/nginx.pid;
error_log /var/log/nginx/error.log;
include /etc/nginx/modules-enabled/*.conf;

events {
	worker_connections 768;
	# multi_accept on;
}

http {

	sendfile on;
	tcp_nopush on;
	types_hash_max_size 2048;
	server_tokens off; # Recommended practice is to turn this off

	include /etc/nginx/mime.types;
	default_type application/octet-stream;

	ssl_protocols TLSv1.2 TLSv1.3; # Dropping SSLv3 (POODLE), TLS 1.0, 1.1
	ssl_prefer_server_ciphers off; # Don't force server cipher order.

	access_log /var/log/nginx/access.log;

	gzip on;


	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
}
```

- `/usr/lib/systemd/system/nginx.service` is read by Linux (`systemd`) to **manage the OS process lifecycle** (how to start, stop, monitor, and auto-restart Nginx).
- `/etc/nginx/nginx.conf` is read by Nginx itself to configure web server behavior (ports, routing, SSL certificates, static files, and reverse proxying to WSGI/uWSGI).
- Any files prefixed with `include` are also referred to get the full configuration.

### Inspect the routing

`/etc/nginx/sites-enabled/` is the most important `include` file that **configures Nginx to act as a Reverse Proxy for your Flask application**.

``` bash
$ ls /etc/nginx/sites-enabled/
flask_app
```

``` bash title="/etc/nginx/sites-enabled/flask_app" hl_lines="3 4 6 7 8 9"
$ cat /etc/nginx/sites-enabled/flask_app
server {
    listen 80;
    server_name _;

    location / {
        include proxy_params;
        proxy_pass http://unix:/var/www/flask_app/flask_app.sock;
        proxy_read_timeout 1s;
    }
}
```

- `listen 80`: Nginx is listening for incoming unencrypted HTTP traffic on Port 80.
- `server_name _;`: `_` is Nginx's wildcard / catch-all hostname, telling Nginx to route any HTTP request to this server block.
- `location / { ... }`: Matches the root path `/` and all sub-paths.
- `include proxy_params;`: Includes standard reverse proxy headers from `/etc/nginx/proxy_params`.
- `proxy_pass ...`: Nginx forwards the incoming HTTP request over a Unix Domain Socket located at `/var/www/flask_app/flask_app.sock`. A WSGI server (such as Gunicorn or uWSGI) running your Flask app must be active and listening on that exact `.sock` file.
- `proxy_read_timeout 1s;`: Nginx will wait at most 1 second for the Flask application to generate and return a response. If your Flask app takes longer than 1 second (e.g., a slow database query, an external API call, or high CPU load), Nginx will immediately terminate the request and return an `HTTP 504 Gateway Timeout` to the user.

### Check Socket File Permission

Since the proxy passes to `/var/www/flask_app/flask_app.sock`, Nginx will throw a `502 Bad Gateway` (or `13: Permission denied in /var/log/nginx/error.log`) if the socket permissions or ownership are misconfigured:

``` bash
$ ls -l /var/www/flask_app/flask_app.sock
srwxrwx--- 1 www-data www-data 0 Sep  6 04:52 /var/www/flask_app/flask_app.sock
```

Checking that `www-data` has read/write access to the socket.

### Test Nginx Config

When debugging Nginx after editing config files, testing syntax is standard practice:

``` bash title="nginx -t"
$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

---

## 3. Inspect uWSGI

To find the systemd service that listens on the socket Nginx forwards the incoming request to, execute `ss -xlp | grep <socket_name>`.

``` bash title="sudo ss -xlp"
$ sudo ss -xlp | grep flask_app.sock
u_str LISTEN 0      2048                 /var/www/flask_app/flask_app.sock 5724            * 0    users:(("gunicorn",pid=1070,fd=5),("gunicorn",pid=711,fd=5))
```

`systemctl status <PID>` will reveal the service:

``` bash title="systemctl status 711" hl_lines="11 12"
$ systemctl status 711
● gunicorn.service - Gunicorn instance to serve Flask
     Loaded: loaded (/etc/systemd/system/gunicorn.service; enabled; preset: enabled)
     Active: active (running) since Sun 2026-09-06 18:35:41 UTC; 28min ago
 Invocation: 6271b1e304754f80bbf9584025ea04b2
   Main PID: 711 (gunicorn)
      Tasks: 2 (limit: 501)
     Memory: 14.3M (peak: 32.7M, swap: 17M, swap peak: 17M)
        CPU: 346ms
     CGroup: /system.slice/gunicorn.service
             ├─ 711 /usr/bin/python3 /usr/bin/gunicorn --workers 1 --bind unix:/var/www/flask_app/flask_app.sock -m 007 app:app
             └─1070 /usr/bin/python3 /usr/bin/gunicorn --workers 1 --bind unix:/var/www/flask_app/flask_app.sock -m 007 app:app

Sep 06 18:35:41 i-0d36b8d4f94e977e0 systemd[1]: Started gunicorn.service - Gunicorn instance to serve Flask.
Sep 06 18:35:42 i-0d36b8d4f94e977e0 gunicorn[711]: [2026-09-06 18:35:42 +0000] [711] [INFO] Starting gunicorn 23.0.0
Sep 06 18:35:42 i-0d36b8d4f94e977e0 gunicorn[711]: [2026-09-06 18:35:42 +0000] [711] [INFO] Listening at: unix:/var/www/flask_app/flask_app.sock (711)
Sep 06 18:35:42 i-0d36b8d4f94e977e0 gunicorn[711]: [2026-09-06 18:35:42 +0000] [711] [INFO] Using worker: sync
Sep 06 18:35:42 i-0d36b8d4f94e977e0 gunicorn[1070]: [2026-09-06 18:35:42 +0000] [1070] [INFO] Booting worker with pid: 1070
```

The `CGroup` (Control Group) line is a Linux kernel mechanism used by `systemd` to track, isolate, and group all processes belonging to this service.
- It displays the Master(`PID 711`) and Worker(`PID 1070`) processes.
- It reveals the exact command line and flags that systemd used to launch Gunicorn.
- It tells you that these processes belong to the `system.slice` cgroup.
  - This is where Linux enforces CPU, Memory, and PID limits (e.g. `Tasks: 2 (limit: 501)` and `Memory: 14.3M`).

In case of uWSGI, you will find the config file(`.ini`) in the `CGroup`

``` bash hl_lines="13"
$ systemctl status 4158
Warning: The unit file, source configuration file or drop-ins of myproject.service changed on disk. Run 'systemctl daemon-reload' to reload units.
● myproject.service - uWSGI instance to serve myproject
     Loaded: loaded (/etc/systemd/system/myproject.service; enabled; preset: enabled)
     Active: active (running) since Wed 2026-09-02 02:14:49 UTC; 4 days ago
 Invocation: 1fd89a2cf8d647d2b059247bae8b6138
   Main PID: 4154 (uwsgi)
      Tasks: 6 (limit: 657)
     Memory: 67.6M (peak: 67.9M)
        CPU: 18.596s
     CGroup: /system.slice/myproject.service
             ├─4154 /home/sammy/myproject/myprojectenv/bin/uwsgi --ini myproject.ini
             ├─4155 /home/sammy/myproject/myprojectenv/bin/uwsgi --ini myproject.ini
             ├─4156 /home/sammy/myproject/myprojectenv/bin/uwsgi --ini myproject.ini
             ├─4157 /home/sammy/myproject/myprojectenv/bin/uwsgi --ini myproject.ini
             ├─4158 /home/sammy/myproject/myprojectenv/bin/uwsgi --ini myproject.ini
             └─4159 /home/sammy/myproject/myprojectenv/bin/uwsgi --ini myproject.ini

Sep 06 13:38:58 ip-172-31-45-111 uwsgi[4155]: [pid: 4155|app: 0|req: 553/1400] 54.163.206.20 () {36 vars in 524 bytes} [Sun Sep  6 13:38:58 2026] GET /favicon.ico => generated 207 bytes in 0 msecs (HTTP/1.1 404) 2 headers in 87 bytes (2 switches on core 0)
Sep 06 13:39:59 ip-172-31-45-111 uwsgi[4157]: [pid: 4157|app: 0|req: 231/1401] 54.89.101.241 () {36 vars in 502 bytes} [Sun Sep  6 13:39:59 2026] GET / => generated 40 bytes in 0 msecs (HTTP/1.1 200) 2 headers in 79 bytes (2 switches on core 0)
```

The `.ini` file contains the uWSGI configurations:

``` bash hl_lines="3 6 8-9"
$ cat /home/sammy/myproject/myproject.ini
[uwsgi]
module = wsgi:app

master = true
processes = 5

socket = myproject.sock
chmod-socket = 660
vacuum = true

die-on-term = true
```

- `module = wsgi:app`: This represents the app entry point, which points to `app` object in `wsgi.py`.
- `processes = 5`: 5 worker uWSGI processes are to be spun up.
- `socket = myproject.sock`: The socket uWSGI is listening on.
- `chmod-socket = 660`: Ensure read and wrtie are permitted on the socket.




---

## 4. Verify Flask & SQLite

### Verify Flask app routing logic

`wsgi.py` will tell you where the `app` object is sourced(`myproject`):

``` bash title="wsgi.py"
$ cat /home/sammy/myproject/wsgi.py
from myproject import app

if __name__ == "__main__":
    app.run()
```

The default route(`/`) is defined in `myproject.py`:

``` bash title="myproject.py"
$ cat ~/myproject/myproject.py
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "<h1 style='color:blue'>Hello There!</h1>"

if __name__ == "__main__":
    app.run(host='0.0.0.0')
```
