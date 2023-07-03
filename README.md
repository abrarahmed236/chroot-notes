# chroot

Notes for using chroot and schroot properly

## Sources

[Logan\'s Note] | [Josiah Ulfers]

[Josiah Ulfers]: <https://josiahulfers.com/2018/03/31/schroot-cheatsheet/>
[Logan\'s Note]: <http://logan.tw/posts/2018/02/24/manage-chroot-environments-with-schroot/>

## To-do

* say i want to create three chroot directories to test three different configurations, create instructions for that
* separate instructions for minimal setup and use case based sections

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

### Prepare a root filesystem

The root folder needs to have a filesystem set up consisting of utility commands, shared libraries, system configuration files etc.

You can set these up manually or you can use a command like `debootstrap` to automate this process.

For example, to create an Ubuntu 16.04 (Xenial Xerus) root directory at `/srv/chroot/xenial`, run the command below:

```bash
sudo debootstrap xenial /srv/chroot/xenial http://archive.ubuntu.com/ubuntu
```

or if you want Ubuntu 20.04 (Focal Fossa) at `/srv/chroot/focal`

```bash
sudo debootstrap focal /srv/chroot/focal http://archive.ubuntu.com/ubuntu
```

If you would like to run a 32-bit chroot on an x86-64 host, run `debootstrap --arch=i386`

```bash
sudo debootstrap --arch=i386 xenial /srv/chroot/xenial32 http://archive.ubuntu.com/ubuntu
```

> Note: add notes about how to add a specific release of ubuntu. If possible.

### Create a config file

After setting up a root file system, we have to create a schroot configuration file in `/etc/schroot/chroot.d/`.

We can easily add, modify, or remove chroot configurations by simply adding or removing files in this directory.

Let's make config files for our xenial based chroot first

```bash
sudo vim /etc/schroot/chroot.d/xenial.conf
```

And populate the configuration (substitute USERNAME with your user name)

```config
[xenial]                        # name, user defined
description=Ubuntu 16.04 Xenial Xerus   # description
directory=/srv/chroot/xenial    # root directory of the chroot environment
root-users=USERNAME             # comma seperated list of users that can get
                                # root privileges by specifying '-u root'
users=USERNAME                  # comma seperated list of users that have 
                                # access to the chroot environment
type=directory                  # either 'directory' or 'plain', plain won't run any
                                # setup scripts, directory does convenient things
                                # like mounting the home directory.
#personality=linux32            # To run a 32-bit chroot environment on a x86-64 host
```

After creating the configuration file, `schroot -l` should list the chroot environments that are available:

```console
$ schroot -l
chroot:xenial
```

> ### Note
>
> Schroot configuration files must be owned by root and must not be writable by other users.  Try `ls -al` in the directory `/etc/schroot/chroot.d/`

## Enter a Chroot Environment

To enter a chroot environment, `run schroot -c [name]`:

```console
$ schroot -c xenial
(xenial)user@hostname:~$
```

To leave a chroot environment, type `exit`:

```console
(xenial)user@hostname:~$ exit
$
```

