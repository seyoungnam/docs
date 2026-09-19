# The rwx Permission Model

## Core Concepts 

``` bash
drwxr-xr--  2  www-data  www-data  4096  Sep 8 12:00  app_data
-rw-r--r--  1  www-data  www-data  1024  Sep 8 12:00  app.db
│└──┬──┘└──┬──┘└──┬──┘
│   │      │      └── Others (read only: 4)
│   │      └───────── Group (read + execute: 4 + 1 = 5)
│   └──────────────── User/Owner (read + write + execute: 4 + 2 + 1 = 7)
└──────────────────── File Type ('d' = directory, '-' = regular file, 's' = socket)
```

### Directory permissions

- **Read (r)**: able to see contents
- **Write (w)**: able to change contents (regardless of file ownership)
- **Execute (x)**: able to enter a directory (using `cd`).

---

## Notation: Symbolic vs Numeric

When you run `ls -l`, you will see a string like `-rwxr-xr--`.

``` bash
ls -la /etc/passwd
-rw-r--r-- 1 root root 2778 Apr 28 22:39 /etc/passwd
```

### Symbolic Notation

The string is divided into three blocks after the initial file type marker:

`[Owner][Group][Others]`

``-rwxr-xr--` indicates:

- `rwx`: Owner can do everything.
- `r-x`: Group can read and execute, but not modify.
- `r--`: Others can only read.

### Numeric (Octal) Notation

This is the most common way to set permissions using `chmod`. It uses a simple math system where each action is assigned a value:

- **Read** = 4(`100` in binary number)
- **Write** = 2(`10` in binary number)
- **Execute** = 1(`1` in binary number)

You add these up to get a single digit for each "Who":

- **7**(4+2+1): Full permissions (`rwx`).
- **6**(4+2): Read and Write (`rw-`).
- **5**(4+1): Read and Execute (`r-x`).
- **4**(4): Read only (`r--`).

!!! Note

    `chmod 755 script.sh` gives the owner full access (7), while the group and others can only read and execute (5).

---

## The Root Exception

The **Root** user(UID 0) is the "God mode" of Linux. Root ignores almost all standard permission checks. Using `sudo` (SuperUser DO) allows you to temporarily assume these powers to bypass the standard permission blocks.

---

## Web Stack & Database Troubleshooting Patterns

### Nginx & Unix Domain Sockets

**Error Code**:

- `502 Bad Gateway`
- `403 Forbidden`

- **Symptom**: Nginx error log reports

    ``` bash
    (13: Permission denied) while connecting to upstream: "unix:/home/sammy/myproject/myproject.sock"
    ```
- **Root Cause**: Nginx worker runs as `www-data`, but the socket was created by `root` or a different service user with `chmod 660`.
- **Fix:**

``` bash
# Check socket ownership and permissions
ls -al unix:/home/sammy/myproject/myproject.sock
# Fix ownership or configure uWSGI with `chmod-socket = 660` and `chown-socket = www-data:www-data`
sudo chown www-data:www-data unix:/home/sammy/myproject/myproject.sock
```

### SQLite Write / WAL Failures

- **Symptom:** Python/Flask throws `sqlite3.OperationalError: attempt to write a readonly database` or `database is locked`.
- **Root Cause:** SQLite needs to create temporary lock and journal files (`.db-wal`, `.db-shm`) in the containing folder. If the directory is owned by `root:root` and lacks write access for the application user (`www-data`), writes will fail even if `chmod 666 app.db` was executed on the file itself.
- **Fix:**

``` bash
# Ensure the application user owns the DB file AND its parent directory
sudo chown -R www-data:www-data /var/www/app/data
sudo chmod 755 /var/www/app/data
sudo chmod 644 /var/www/app/data/app.db
```

---

## Frequently used commands

- `sudo su - abe`: switch to user `abe` with a login shell and environment
- `sudo chmod 0644 project_*`: set permissions of `project_*` to `0644` (`rw-r--r--`: read/write for owner, read-only for group and others)

``` bash title="sudo chmod 0644 project_*" hl_lines="6-9 16-19"
admin@i-0337ba423669056e8:~/shared$ ls -al
total 28
drwxr-xr-x 2 admin  admin  4096 Feb  2  2025 .
drwxr-xr-x 6 admin  admin  4096 Sep  8 21:02 ..
-rw-r--r-- 1 root   admin    38 Feb  2  2025 ALL
-rw-r----- 1 abe    abe      27 Feb  2  2025 project_abe
-rw-r----- 1 betty  betty    29 Feb  2  2025 project_betty
-rw-r----- 1 carlos carlos   30 Feb  2  2025 project_carlos
-rw-r----- 1 debora debora   30 Feb  2  2025 project_debora
admin@i-0337ba423669056e8:~/shared$ sudo chmod 0644 project_*
admin@i-0337ba423669056e8:~/shared$ ls -al
total 28
drwxr-xr-x 2 admin  admin  4096 Feb  2  2025 .
drwxr-xr-x 6 admin  admin  4096 Sep  8 21:02 ..
-rw-r--r-- 1 root   admin    38 Feb  2  2025 ALL
-rw-r--r-- 1 abe    abe      27 Feb  2  2025 project_abe
-rw-r--r-- 1 betty  betty    29 Feb  2  2025 project_betty
-rw-r--r-- 1 carlos carlos   30 Feb  2  2025 project_carlos
-rw-r--r-- 1 debora debora   30 Feb  2  2025 project_debora
```

- `sudo groupadd project`: create a new group called `project`
- `sudo usermod -aG project abe`: add user `abe` to the `project` group
- `sudo chown root:project <file>`: change the owner of `<file>` to `root` and its group to `project`

``` bash title="sudo chown root:project ALL" hl_lines="10 20"
admin@i-0337ba423669056e8:~/shared$ sudo groupadd project
admin@i-0337ba423669056e8:~/shared$ sudo usermod -aG project abe
admin@i-0337ba423669056e8:~/shared$ sudo usermod -aG project betty
admin@i-0337ba423669056e8:~/shared$ sudo usermod -aG project carlos
admin@i-0337ba423669056e8:~/shared$ sudo usermod -aG project debora
admin@i-0337ba423669056e8:~/shared$ ls -al
total 28
drwxr-xr-x 2 admin  admin  4096 Feb  2  2025 .
drwxr-xr-x 6 admin  admin  4096 Sep  8 21:02 ..
-rw-r--r-- 1 root   admin    38 Feb  2  2025 ALL
-rw-r--r-- 1 abe    abe      27 Feb  2  2025 project_abe
-rw-r--r-- 1 betty  betty    29 Feb  2  2025 project_betty
-rw-r--r-- 1 carlos carlos   30 Feb  2  2025 project_carlos
-rw-r--r-- 1 debora debora   30 Feb  2  2025 project_debora
admin@i-0337ba423669056e8:~/shared$ sudo chown root:project ALL
admin@i-0337ba423669056e8:~/shared$ ls -al
total 28
drwxr-xr-x 2 admin  admin   4096 Feb  2  2025 .
drwxr-xr-x 6 admin  admin   4096 Sep  8 21:02 ..
-rw-r--r-- 1 root   project   38 Feb  2  2025 ALL
-rw-r--r-- 1 abe    abe       27 Feb  2  2025 project_abe
-rw-r--r-- 1 betty  betty     29 Feb  2  2025 project_betty
-rw-r--r-- 1 carlos carlos    30 Feb  2  2025 project_carlos
-rw-r--r-- 1 debora debora    30 Feb  2  2025 project_debora
```

- `sudo chattr +a <file>`: set the append-only attribute on `<file>` (data can only be appended; cannot be overwritten, truncated, renamed, or deleted, even by root)
- `sudo chattr -a <file>`: remove the append-only attribute from `<file>`
- `lsattr <file>`: list file attributes to verify flags (e.g. `a` for append-only)

- `sudo -l`: check if the current user has `sudo` privileges
- `id`: check the current user's identity

``` bash
$ id
uid=1000(admin) gid=1000(admin) groups=1000(admin),4(adm),20(dialout),24(cdrom),25(floppy),27(sudo),29(audio),30(dip),44(video),46(plugdev),989(docker)
```

- `docker run --rm -v /:/mnt alpine cat /mnt/root/mysecret.txt`: 
    - `docker run`: creates and starts a new container.
    - `--rm`: automatically cleans up and removes the container once it exits.
    - `-v /:/mnt`: **bind mounts** the host machine's root filesystem (`/`) into the container at `/mnt`.
    - `alpine`: Uses the lightweight Alpine Linux[1] image to run the command.
    - `cat /mnt/root/mysecret.txt`: Reads the file at `/mnt/root/mysecret.txt` (which maps directly to the host's `/root/mysecret.txt`)


---

## SadServer Questions

- **Yokohama** (Linux users working together): Configure shared directory permissions, SGID bits, and group collaboration access.
- **Nuuk** (More SSH Troubles): Fix SSH key configuration, file permissions on ~/.ssh/ and authorized_keys, or SSH daemon settings.
- **Pokhara** (SSH and other sshenanigans): Multi-user SSH configuration, local loopback key authentication, and sudo privileges.
- **Annapurna** (High Privileges): Sudoers configuration (/etc/sudoers) and privilege boundaries.
- **Taipei** (Come a-knocking): Port-knocking sequences and firewall rules (iptables / ufw).