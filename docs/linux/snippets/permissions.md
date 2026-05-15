# File Permissions

At its core, Linux permissions are the "security guards" that determine who can interact with the files and directories on your system. Everything in Linux follows a **Three-by-Three** logic: three types of people and three types of actions.

---

## Three by Three

### The Three Whos(Ownership)

Every file and directory is assigned to three distinct entities:

- **User (u)**: The individual account that owns the file (usually the person who created it).
- **Group (g)**: A collection of users who share specific access rights.
- **Others (o)**: Everyone else on the system who isn't the owner or in the group.

### The Tree Whats(Actions)

There are three basic actions you can perform:

- **Read (r)**: View the contents of a file or list the files in a directory.
- **Write (w)**: Modify a file or add/delete files within a directory.
- **Execute (x)**: Run a file as a program/script or "enter" a directory (using `cd`).

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

## Permissions in Networking(Socket)

It's worth noting that **Sockets are treated like files**.

- If your Go web server tries to bind to a "privileged" port (anything below 1024), the kernel checks permissions.
- Usually, only a process with Root privileges (or specific capabilities) can open those ports.