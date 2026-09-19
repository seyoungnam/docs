# Containers From Scratch

---

## Basics

### Example

If `docker run ubuntu`, get inside, and look up processes(`ps`), we only see a couple of PIDs.

``` bash
$ sudo docker run --rm -it ubuntu /bin/bash
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
6a7c4f6d8c38: Pull complete 
08f5f5b2a2b0: Pull complete 
c04a683f373c: Download complete 
Digest: sha256:513c074113a871b51a8d16ab445c88779d6452d937a164fb5cc479f32668a41d
Status: Downloaded newer image for ubuntu:latest
root@b478d4fefd64:/# hostname
b478d4fefd64
root@b478d4fefd64:/# ps
    PID TTY          TIME CMD
      1 pts/0    00:00:00 bash
     10 pts/0    00:00:00 ps
```

The PIDs seen in the `ubuntu` container are different from those in the host linux.

``` bash
$ ps
    PID TTY          TIME CMD
  52973 pts/5    00:00:00 bash
  53065 pts/5    00:00:00 ps
```

### Namespaces

The namespace is restricting the view of the process has of things that are going on on the host machine. We set up these namespaces using syscalls.

---

## Reference

<iframe width="560" height="315" src="https://www.youtube.com/embed/8fi7uSYlOdc?si=A9pQwMScoNy1QNd_" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>