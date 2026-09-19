# Linux File System

- `/bin`: contains binaries or executables that are essential to the entire operating system(e.g., `gzip`, `curl`, `ls`).
- `/sbin`: contains system binaries that should be executed by the super user (root) like `mount` or `deluser`
- `/lib`: binaries may share common libraries stored in the `lib` directory 
- `/usr`: contains non-essential installed binaries and applications to the operating system, with its own `bin` and `sbin` directories
    - `/usr/local`: contains any binaries that you compile manually to provide a safe place that won't conflict with any software installed by a system package manager. All these binaries get mapped together with the `PATH` env var. 
- `/etc`: stands for **Editable Text Configuration**. Many of these files end in `.conf`. They are typically just text-based config files.
- `/home`: contains folders named after each linux user. It contains the files, configurations, and software for that user. You need to be logged in as that user or as a root user to modify it. 
- `/boot`: contains the files needed to boot the system like the linux kernel itself. 
- `/dev`: stands for device files. You can interface with hardware or drivers as if they were regular files. 
- `/opt`: contains optional or add-on software 
- `/var`: contains variable files that will change as the operating system is being used. Things like logs and cache files.
- `/tmp`: for temporary files that won't be persisted between reboots
- `/proc`: an illusinary file system that doesn't actually exist on the dis but is created in memory on the fly by the linux kernel to keep track of runing processes.

