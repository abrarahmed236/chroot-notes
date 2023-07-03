# chroot

Notes for using chroot and schroot properly

## Sources

[Logan\'s Note] | [Josiah Ulfers]

[Josiah Ulfers]: <https://josiahulfers.com/2018/03/31/schroot-cheatsheet/>
[Logan\'s Note]: <http://logan.tw/posts/2018/02/24/manage-chroot-environments-with-schroot/>

## To-do

* what is the difference between schroot and chroot ?
* say i want to create three chroot directories to test three different configurations, create instructions for that
* separate instructions for minimal setup and use case based sections
* can you and should you run multiple sessoins of the same schroot environment ?
* you can start multiple sessions of the schroot environment.
  * what is the use of this ?

## Installation

```bash
sudo apt install schroot debootstrap
```

## Setup

Choose a location for creating the root directories for chroot.

```bash
/srv    # is a reasonable location
/var    # another reasonable location
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

To execute a command in a chroot and leave a chroot after the command completes, append `--` and the command to be executed. For example, to execute the `cat /etc/lsb-release` command in the chroot environment, run the command below:

```console
$ schroot -c xenial -- cat /etc/lsb-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=16.04
DISTRIB_CODENAME=xenial
DISTRIB_DESCRIPTION="Ubuntu 16.04 LTS"
$
```

To enter a chroot with root privileges, append -u root to the command line:

```console
$ schroot -c xenial -u root
(xenial)root@hostname:/home/user#
```

## Manage Sessions (revisit)

The simpliest `schroot -c` command mentioned in the previous section starts an automatic session before running the command and ends the automatic session after the command completes. When a session starts, `schroot` will mount the file systems and execute the start hooks. When a session ends, `schroot` will execute the exit hooks and unmount the file systems. It may be time consuming if you have to enter and leave a chroot several times.

Under some circumstances (e.g. running several unit tests with `schroot`), it would be great to manage sessions manually. This section covers the command to start a session, list sessions, enter a session, and clean up a session.

### Start a Schroot Session

To start a `schroot` session, run `schroot -b -c [name]`:

```console
$ schroot -b -c xenial
xenial-05735790-a1ab-464b-8675-4e66111370d3
```

The `schroot` command will print the session name to the standard output. To keep the session name in an environment variable, run this command instead:

```console
SESSION="$(schroot -b -c xenial)"
```

You may specify a session name with `-n [session-name]` as well:

```console
$ schroot -b -c xenial -n mysession
mysession
```

### List all Schroot Sessoins

To list all schroot sessions, run schroot -l --all-sessions:

```console
$ schroot -l --all-sessions
session:xenial-05735790-a1ab-464b-8675-4e66111370d3
```

### Enter a Schroot Session

To enter a `schroot` session, run `schroot -r -c [session-name]`:

```console
$ schroot -r -c xenial-05735790-a1ab-464b-8675-4e66111370d3
(xenial)user@hostname:~$
```

This is similar to the simpliest `schroot -c` command mentioned earlier. You may append command to be executed or specify `-u root` for root privileges.

### End a Schroot Session

To end a schroot session (i.e. unmount all file systems and execute the exit hooks), `run schroot -e -c [session-name]`:

```console
schroot -e -c xenial-05735790-a1ab-464b-8675-4e66111370d3
```

BTW, if the computer reboots before the simpliest `schroot -c` completes, the automatic session won't be ended. To clean up those sessions, you have to find their names with `schroot -l --all-sessions` and end those sessions with `schroot -e -c [session-name]`.
