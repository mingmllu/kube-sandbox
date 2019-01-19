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

```
$ sudo mkdir /var/nfs/general -p
```
NFS will translate any root operations on the client to the nobody:nogroup credentials as a security measure. Therefore, we need to change the directory ownership to match those credentials.

```
$ sudo chown nobody:nogroup /var/nfs/general
```

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

We’re using the same configuration options for both directories with the exception of no_root_squash. Let’s take a look at what each one means.

* **rw**: This option gives the client computer both read and write access to the volume.
* **sync**: This option forces NFS to write changes to disk before replying. This results in a more stable and consistent environment since the reply reflects the actual state of the remote volume. However, it also reduces the speed of file operations.
* **no_subtree_check**: This option prevents subtree checking, which is a process where the host must check whether the file is actually still available in the exported tree for every request. This can cause many problems when a file is renamed while the client has it opened. In almost all cases, it is better to disable subtree checking.
* **no_root_squash**: By default, NFS translates requests from a root user remotely into a non-privileged user on the server. This was intended as security feature to prevent a root account on the client from using the file system of the host as root. no_root_squash disables this behavior for certain shares.

To make the shares available to the clients that you configured, restart the NFS server with the following command
```
$ sudo systemctl restart nfs-kernel-server
```

### Step 4 — Adjusting the Firewall on the Host

```
$ sudo ufw status
Status: inactive
```

### Step 5 — Creating the Mount Points on the Client

In order to make the remote shares available on the client, we need to mount the host directory on an empty client directory.

> Note: If there are files and directories in your mount point, as soon as you mount the NFS share, they’ll be hidden. 
> Be sure if you mount in a directory that already exists that the directory is empty.

We’ll create two directories for our mounts:
```
$ sudo mkdir -p /nfs/general
$ sudo mkdir -p /nfs/home
```

### Step 6 — Mounting the Directories on the Client

```
sudo mount 10.0.0.12:/var/nfs/general /nfs/general
sudo mount 10.0.0.12:/home /nfs/home
```

These commands should mount the shares from the host computer onto the client machine. You can double-check that they mounted successfully in several ways. You can check this with a plain ```mount``` or ```findmnt``` command, but ```df -h``` will give you more human readable output illustrates how disk usage is displayed differently for the nfs shares:

```
$ df -h
```

To see how much space is actually being used under each mount point, use the disk usage command ```du``` and the path of the mount. The ```-s``` flag will provide a summary of usage rather than displaying the usage for every file. The ```-h``` will print human readable output.

### Step 7 — Testing NFS Access

### Step 8 — Mounting the Remote NFS Directories at Boot

We can mount the remote NFS shares automatically at boot by adding them to /etc/fstab file on the client.

```
$ sudo nano /etc/fstab
```

At the bottom of the file, we're going to add a line for each of our shares. They will look like this:

```
10.0.0.12:/var/nfs/general    /nfs/general   nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0
10.0.0.12:/home       /nfs/home      nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0
```
The client server will automatically mount the remote partitions at boot, although it may take a few moments for the connection to be made and the shares to be available.

### Step 9 — Unmounting an NFS Remote Share

```
$ cd ~
$ sudo umount /nfs/home
$ sudo umount /nfs/general
```
This will remove the remote shares, leaving only your local storage accessible:

```
$ df -h
```


