---
layout: post
status: publish
published: true
title: Proxmox home cluster (II)
author: Manuel Molina
about_author: "/us/manuel/readme.html"
author_email: mmc@pocosmhz.org
date: '2025-04-15 18:10:00 +0100'
tags:
- hypervisor
- home
- ha
- budget
comments: true
---
This is the second part of my [previous post]({% post_url 2025-04-13-proxmox-home-cluster-i %}) about Proxmox basic installation.

Now we have a working cluster, where we can start virtual hosts and [LXC](https://en.wikipedia.org/wiki/LXC) containers. However, the storage is local to each node.

What we'll achieve next is to incorporate shared storage to this cluster.

As a starting point, I used [this blog post from Lobobrothers](https://tech.lobobrothers.com/proxmox-y-ceph-de-0-a-100-parte-iii/) (it's in Spanish).

The steps I followed:

0. If you installed Proxmox with a single disk, and left enough disk space in the installation process (see previous post), **here is the time to partition that space** to make it available for Ceph:

    See the current configuration:
    ```bash
    root@pve01:~# fdisk /dev/nvme0n1

    Welcome to fdisk (util-linux 2.38.1).
    Changes will remain in memory only, until you decide to write them.
    Be careful before using the write command.

    This disk is currently in use - repartitioning is probably a bad idea.
    It's recommended to umount all file systems, and swapoff all swap
    partitions on this disk.


    Command (m for help): p

    Disk /dev/nvme0n1: 931.51 GiB, 1000204886016 bytes, 1953525168 sectors
    Disk model: KINGSTON SNV3S1000G                     
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: gpt
    Disk identifier: 712DDA48-E2CF-4783-AEAD-5103BB5DAD17

    Device             Start        End    Sectors   Size Type
    /dev/nvme0n1p1        34       2047       2014  1007K BIOS boot
    /dev/nvme0n1p2      2048    2099199    2097152     1G EFI System
    /dev/nvme0n1p3   2099200  421529599  419430400   200G Linux LVM

    Command (m for help): 
    ```
    Create a new partition:
    ```bash
    Command (m for help): n
    Partition number (4-128, default 4): 
    First sector (2099200-421529599, default 2099200): 
    Last sector, +/-sectors or +/-size{K,M,G,T,P} (2099200-421529599, default 421529599): 

    Created a new partition 4 of type 'Linux filesystem' and of size 730.5 GiB.

    Command (m for help): w
    The partition table has been altered.
    Syncing disks.
    ```

1. After loggin in to the admin console, I click on Datacenter -> Ceph. And we receive this message:
![Ceph not installed warning](/content/images/2025-04-15-proxmox-home-cluster-ii/ceph-not-installed.png)
So we accept.

2. We're offered a screen to select which version to install:
![Ceph install selection](/content/images/2025-04-15-proxmox-home-cluster-ii/ceph-install-selection.jpg)
After selecting the latest non-subscription version, we click on *Start squid installation*.

3. Once we finish the installation, click on *Next*.
Select the network for Ceph to use:
![Ceph configuration selection](/content/images/2025-04-15-proxmox-home-cluster-ii/ceph-config.png)

4. Installation of Ceph in the first node is done.
![Ceph configuration selection](/content/images/2025-04-15-proxmox-home-cluster-ii/ceph-finished.png)

5. As we've been told, we'll repeat the steps in the nodes `pve02` and `pve03`.

6. Now, for each of the nodes, we click on `Ceph` -> `OSD` -> `Create: OSD`:
![Ceph create OSD](/content/images/2025-04-15-proxmox-home-cluster-ii/ceph-create-osd.png)

7. Again, for each of the nodes, we click on `Ceph` -> `Monitor` -> `Create`:
![Ceph create Monitor](/content/images/2025-04-15-proxmox-home-cluster-ii/ceph-create-monitor.png)

8. For nodes `pve02` and `pve03` we'll create additional manager processes, by clicking on `Ceph` -> `Monitor` -> `Manager` -> `Create`:
![Ceph create additional manager](/content/images/2025-04-15-proxmox-home-cluster-ii/ceph-create-manager.png)

9. Now, if you were hosting different storage types, you would want to establish a new set of CRUSH rules for the OSDs. If you have a single storage space per node and OSD, you can safely skip this step.

In a case you wanted to have Ceph with different set of disks per node, like spinning, SSD, or a bunch of disks to be split in different groups, please go ahead and check [Ceph CRUSH & Device Classes](https://pve.proxmox.com/wiki/Deploy_Hyper-Converged_Ceph_Cluster#pve_ceph_device_classes) section of the documentation.

10. If we now click on `Ceph` -> `OSD` we can see something like this:
![Ceph OSD list](/content/images/2025-04-15-proxmox-home-cluster-ii/ceph-osd.png)

11. Now it's time for create a Ceph pool, where virtual machines will be stored. Go ahead and click on `Ceph` -> `Pools` -> `Create`:
![Ceph create pool](/content/images/2025-04-15-proxmox-home-cluster-ii/ceph-pool1.png)

12. We can check under `Datacenter` -> `Storage` that we do have `pool1` available for shared storage:
![Ceph shared storage](/content/images/2025-04-15-proxmox-home-cluster-ii/ceph-shared.png)

Now we're ready to create a new virtual machine in the Ceph shared storage.