---
title: "chroot, namespace, & cgroups"
full_title: "Lesson 2: chroot, namespaces, cgroups"
weight: 2
date: 2020-10-23T1:00:00-00:97
draft: true
---

There's no magic here. Getting back to first principles. A container is: chroot, namespaces, & cgroups. With the exception of cgroups, these are technologies that have been around for a longtime.

* **chroot**: A chroot is an operation that changes the apparent root directory for the current running process and their children. A program that is run in such a modified environment cannot access files and commands outside that environmental directory tree
* **namespaces**: Namespaces are a feature of the Linux kernel that partitions kernel resources such that one set of processes sees one set of resources while another set of processes sees a different set of resources. The feature works by having the same namespace for a set of resources and processes, but those namespaces refer to distinct resources. Resources may exist in multiple spaces. Examples of such resources are process IDs, hostnames, user IDs, file names, and some names associated with network access, and interprocess communication.
* **cgroups**: cgroups is a Linux kernel feature that limits, accounts for, and isolates the resource usage of a collection of processes. Engineers at Google started the work on this feature in 2006 under the name "process containers"

### Learning Objectives
* *Understand* how linux implements containers and how you can do them yourself.
* *Gain* deeper insight into what docker does for us
* *See* how three core concepts central to containers actually work.


### chroot walk through

Launch your cloud vm and run a priviledged container on your machine. Yes, it's turtles all the way down. We're going to login to linux, run a linux container, and create a container by hand. :)

Launching a priviledged container:
```bash
docker run -it --name docker-host --dns 8.8.8.8 --rm --privileged ubuntu:bionic
```

NOTE: `--dns 8.8.8.8` is only because the dns doesn't always seem to resolve on the Amazon VDI. If you're running locally, you can safely skip this.

Docker privileged mode grants a Docker container root capabilities to all devices on the host system. Running a container in privileged mode gives it the capabilities of its host machine. For example, it enables it to modify App Arm and SELinux configurations.

With the host’s kernel features and device access, you can even install a new instance of the Docker platform within the privileged container. Essentially, this mode allows running Docker inside Docker.


Create new root directory
```bash 
mkdir /test-root
chroot /test-root /bin/bash  #This fails because we need to add bash and it's dependences to the linux jail
mkdir /test-root/bin
cp /bin/bash /test-root/bin/bash
ldd /bin/bash
```
`ldd` is a *nix utility that prints the shared libraries required by each program or shared library specified on the command line. Everything (not at the kernel level) running within the linux jail (chroot) needs it's own dependencies brought in. So bash, needs to have it's dependencies available within the jail. ldd shows us it's dependencies (think of these *.so as dll's on windows). 

Now let's add them to the new file system we're creating.
```bash 
mkdir /test-root/lib
mkdir /test-root/lib64

cp /lib/x86_64-linux-gnu/libtinfo.so.5 /lib/x86_64-linux-gnu/libdl.so.2 /lib/x86_64-linux-gnu/libc.so.6 /test-root/lib
cp /lib64/ld-linux-x86-64.so.2 /test-root/lib64

chroot /test-root /bin/bash
```

`exit` will take you out of the `chroot` Make sure you can use `ls` and you can see `/test-root` If you don't see this folder, you are still in the chroot. If you exit too many times, you'll be back at the Amazon Linux instance. Another thing you can look at is `cat /etc/issue` You should see: `Ubuntu 18.04.5 LTS \n \l` It won't work if you're in the linux jail.

Now it should work, but other commands... like `ls` won't. Why? Let's fix this:
```bash
ldd /bin/ls
```
You can ldd ls and do the same thing to get other commands like `ls` or `cat` to work.


### namespace walk through

debootstrap: debootstrap is a tool which will install a Debian base system into a subdirectory of another, already installed system. It doesn't require an installation CD, just access to a Debian repository.
https://linux.die.net/man/8/debootstrap

Launch a priviledged container (or reuse the one you already had):
```bash
docker run -it --name docker-host --dns 8.8.8.8 --rm --privileged ubuntu:bionic
```

NOTE: `--dns 8.8.8.8` is only because the dns doesn't always seem to resolve on the Amazon VDI. If you're running locally, you can safely skip this.

install debootstrap
```bash
apt-get update -y
apt-get install debootstrap -y
```

setup a basic debian filesystem
```bash
debootstrap --variant=minbase bionic /better-root
```

Namespaces are a feature of the Linux kernel that partitions kernel resources such that one set of processes sees one set of resources while another set of processes sees a different set of resources. The feature works by having the same namespace for a set of resources and processes, but those namespaces refer to distinct resources. Resources may exist in multiple spaces. Examples of such resources are process IDs, hostnames, user IDs, file names, and some names associated with network access, and interprocess communication.

Namespaces are a fundamental aspect of containers on Linux.

The term "namespace" is often used for a type of namespace (e.g. process ID) as well as for a particular space of names.

A Linux system starts out with a single namespace of each type, used by all processes. Processes can create additional namespaces and join different namespaces.

Linux Namespaces originated in 2002 in the 2.4.19 kernel with work on the mount namespace kind. Additional namespaces were added beginning in 2006.

Since kernel version 5.6, there are 8 kinds of namespaces. Namespace functionality is the same across all kinds: each process is associated with a namespace and can only see or use the resources associated with that namespace, and descendant namespaces where applicable. This way each process (or process group thereof) can have a unique view on the resources. Which resource is isolated depends on the kind of namespace that has been created for a given process group.
* Mount (mnt)
* Process ID (pid)
* Network (net)
* Interprocess Communication (ipc)
* UTS
* User ID (user)
* Control group (cgroup) Namespace
* Time Namespace

Let's remove some of the things this file system can see and launch chroot
```bash
#unmount namespaces
unshare --mount --uts --ipc --net --pid --fork --user --map-root-user chroot /better-root bash
```

`ps aux` will not work. There are some things we need to mount for these to work correctly.
```bash
mount -t proc proc /proc
mount -t sysfs sysfs /sys
mount -t tmpfs tmpfs /tmp
```

Compare processes in the chroot and out. Open and `exec bash` on the shell that you're running chroot into. Look at the processes it can see. Look at the pids and tail -f a file and examine it between the two different terminals.

NOTE: Don't get confused between terminals. Look for the 'better-root` to know where you are. If you see better-root, you're in your chroot'd env.
```bash
docker ps
docker exec docker-host bash
```

```bash
 docker exec --help
 ```
```
Usage:	docker exec [OPTIONS] CONTAINER COMMAND [ARG...]

Run a command in a running container

Options:
  -d, --detach               Detached mode: run command in the background
      --detach-keys string   Override the key sequence for detaching a container
  -e, --env list             Set environment variables
  -i, --interactive          Keep STDIN open even if not attached
      --privileged           Give extended privileges to the command
  -t, --tty                  Allocate a pseudo-TTY
  -u, --user string          Username or UID (format: <name|uid>[:<group|gid>])
  -w, --workdir string       Working directory inside the container
```

### cgroup walk through

cgroups (abbreviated from control groups) is a Linux kernel feature that limits, accounts for, and isolates the resource usage (CPU, memory, disk I/O, network, etc.) of a collection of processes.

Engineers at Google (primarily Paul Menage and Rohit Seth) started the work on this feature in 2006 under the name "process containers". In late 2007, the nomenclature changed to "control groups" to avoid confusion caused by multiple meanings of the term "container" in the Linux kernel context, and the control groups functionality was merged into the Linux kernel mainline in kernel version 2.6.24, which was released in January 2008.

Two versions... version two is a total rewrite that discriminates between processes, not threads like v1. It appeared in the linux kernel in 2016.

Exit the chroot and go back to the priveledged container (you should be able to see better-root). Don't go all the way back to the host OS. Otherwise, you'll have to reset up the container.

Install tools, create a control group, and test isolation. XXXX is the pid of your chroot env. Look for the chroot command. It will be the bash pid, right under it.

You'll install this in the first Ubuntu priveledged container that is currently running the unshare process. Check `ps aux` to make sure you're in the right place.
```bash
apt-get install -y cgroup-tools htop
cgcreate -g cpu,memory,blkio,devices,freezer:/sandbox
cgclassify -g cpu,memory,blkio,devices,freezer:sandbox XXXX

#look at the settings, -1 is not seth
cat /sys/fs/cgroup/cpu/sandbox/cpu.cfs_quota_us

# Limit usage at 5% for a multi core system
cgset -r cpu.cfs_period_us=100000 -r cpu.cfs_quota_us=$[ 5000 * $(getconf _NPROCESSORS_ONLN) ] sandbox

#look at the settings
cat /sys/fs/cgroup/cpu/sandbox/cpu.cfs_quota_us
```

inside and then outside the cgrouped chroot environment... let's peg CPU and watch it in htop
```bash
yes docker rocks >> /dev/null
```


