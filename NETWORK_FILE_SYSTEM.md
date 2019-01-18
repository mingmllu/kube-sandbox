## [Set Up an NFS Mount on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nfs-mount-on-ubuntu-16-04)

### Prerequisites

* Two Ubuntu 16.04 servers, each with a non-root user with sudo privileges and private networking enabled.
  * The server that shares its directories as the host (Host IP address: 10.0.0.12)
  * The server that mounts these directories as the client (Client IP address: 10.0.0.18)
 
### Step 1 — Downloading and Installing the Components

#### On the Host

```
$ sudo apt-get update
$ sudo apt-get install nfs-kernel-server
```

#### On the Client

```
$ sudo apt-get update
$ sudo apt-get install nfs-common
```
The package "nfs-common" provides NFS functionality without including unneeded server components. 

### Step 2 — Creating the Share Directories on the Host

NFS-mounted directories are not part of the system on which they are mounted, so by default, the NFS server refuses to perform 
operations that require superuser privileges. This default restriction means that superusers on the client cannot write files 
as root, re-assign ownership, or perform any other superuser tasks on the NFS mount.

### Step 3 — Configuring the NFS Exports on the Host Server

Open the /etc/exports file in your text editor with root privileges:
```
$ sudo nano /etc/exports
```
Add the following lines to the file ```/etc/exports```:
```
/var/nfs/general    10.0.0.18(rw,sync,no_subtree_check)
/home       10.0.0.18(rw,sync,no_root_squash,no_subtree_check)
```
