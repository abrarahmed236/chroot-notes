# chroot

Notes for using chroot and schroot properly

## Installation

```bash
sudo apt install schroot debootstrap
```

## Setup

Choose a location for creating the root directories for chroot.

```bash
/srv    # is a reasonable location
/var    # is sometimes used
/chroot # is also used as a dedicated space
```

We will be using `/srv/chroot`, so we need to make that directory before we proceed

```bash
mkdir -p /srv/chroot
```

### Prepare a root filesystem

The root folder needs to have a filesystem setup consisting of utility commands, shared libraries, system configuration files etc.

You can set these up manually or you can use a command like `debootstrap` to automate this process.

For example, to create an Ubuntu 16.04 (Xenial Xerus) root directory at `/var/chroot/xenial`, run the command below:

```bash
sudo debootstrap xenial /var/chroot/xenial \
    http://archive.ubuntu.com/ubuntu
```

If you would like to run a 32-bit chroot on an x86-64 host, run `debootstrap --arch=i386`

```bash
sudo debootstrap --arch=i386 xenial /var/chroot/xenial32 \
    http://archive.ubuntu.com/ubuntu
```

### Create a config file
