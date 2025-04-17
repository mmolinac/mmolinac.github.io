---
layout: post
status: publish
published: true
title: Proxmox home cluster (I)
author: Manuel Molina
about_author: "/us/manuel/readme.html"
author_email: mmc@pocosmhz.org
date: '2025-04-13 20:10:00 +0100'
tags:
- hypervisor
- home
- ha
- budget
comments: true
---
For some time I've been wondering the best way of having a steady solution to use as a home lab.
That involves a whole lot of factors to think about. Many of them are cost-related.

Nowadays you can think about having a small cluster in your cloud provider of choice.
Even if you automate the creation and destroy of such lab through [IaC](https://en.wikipedia.org/wiki/Infrastructure_as_code), you'll incur in costs that might end being a real burden.

What about a dedicated server? Even the cheapest ones are either not big enough or you have to pay extra to have proper storage.

And what about using real hardware and hosting it at home? Well, I checked some second-hand carrier-grade hardware providers and they offer you really affordable solutions. See [Give1life](https://www.give1life.com), [Jet Computer](https://www.jetcomputer.net) or [Serverando](https://serverando.de/), but there are many others.

As familiar as I am with the hosting and datacenter world, there are three limiting factors in this solution: noise, heat and electric power. Yes, they're affordable and reliable. But you'll need a room for that purpose. As I don't plan on having one, let's check another solution that came to me.

# Configuration selection
## Compute platform
There is a new trend of small factor PCs, called [mini PC](https://en.wikipedia.org/wiki/Mini_PC) that are very handy when you lack proper desktop space. Not only they're small, but also very energy efficient.

For a small desktop PC to browse the web and do basic daily chores, they were fine. However, right now, you can find a very wide variety of options. Some of them are not so *basic*.

Some friends talked me into checking the [Intel N100](https://www.intel.com/content/www/us/en/products/sku/231803/intel-processor-n100-6m-cache-up-to-3-40-ghz/specifications.html) CPU from Intel's 12th gen [Alder Lake](https://en.wikipedia.org/wiki/Alder_Lake#Alder_Lake-N) architecture. Indeed, very power savvy (between 6 and 12 watt), but also very capable (4 CPU cores up to 3.4 GHz).

Theoretically they're able to handle up to 16 GBytes of RAM. That is more than enough for a low cost PC. You can even use it as a workstation, if you stretch the concept a bit.

They made me be interested and I already put my eyes on the [Texhoo QN10](https://www.aliexpress.com/item/1005006445443589.html) mini PC. Please **do not** be mislead by the QN10 SE model.

![TexHoo QN10](/content/images/2025-04-13-proxmox-home-cluster-i/texhoo-qn10.png)

As I was about to settle with the idea of a 16 GB RAM configuration, with dual 2.5 Gbit ethernet interface, multiple USB ports, WIFI6 etcetera, I bumped into [this blog post](https://diycraic.com/2024/12/27/intel-n100-mini-pc-32gb-ram-as-a-home-server/) stating that **there was a stable 32 GBytes RAM configuration**.

For every cluster node, I bought one [Crucial RAM DDR5 32GB 5600MHz SODIMM CL46 - CT32G56C46S5](https://www.amazon.es/dp/B0BLTDTD86) module, as suggested by [Stepyon](https://diycraic.com/user/stepyon/) in the blog, and one [Kingston NV3 NVMe PCIe 4.0 SSD Internal 1TB M.2 2280-SNV3S/1000G](https://www.amazon.es/dp/B0DBR3DZWG) , which have a reasonable price-quality ratio.

Remember that you have to explicitly enable the virtualization options in your BIOS. I also recommend to set up power management so the mini-pc returns to previous state after a power outage.

## Networking
Let's select some interconnect hardware on a budget.
I opted for a non-managed solution that has passive heat sink: [Tenda TEM2010X 8 x 2.5Gbit port switch](https://www.amazon.es/dp/B09M2RXCVN).

# Proxmox installation

## Software installation
In order to install Proxmox we selected version 8.3.1 ISO and downloaded it from their [downloads page](https://www.proxmox.com/en/downloads).

It is very straight-forward, but I'll go quickly through the steps for a standalone installation here:

- First, you can settle with the standard installation option:

![Proxmox installation menu](/content/images/2025-04-13-proxmox-home-cluster-i/proxmox-install-menu.jpg)

- Then you're asked to accept the license:

![Proxmox license](/content/images/2025-04-13-proxmox-home-cluster-i/proxmox-license.jpg)

- Now you are presented with the Proxmox HD selection dialog:

![Proxmox target HD selection](/content/images/2025-04-13-proxmox-home-cluster-i/proxmox-target-hd.jpg)
  - Here you have two options:
  1. If you either don't plan on creating a Ceph cluster, or you plan to do it with a secondary disk, just click on OK and leave the default options (ext4 FS).
  2. If you want to use part of the only disk we have for Ceph, please click on `Options` and reduce the size as stated below:
  ![Proxmox target HD resize](/content/images/2025-04-13-proxmox-home-cluster-i/proxmox-target-hd-resize.jpg)
  
    Now accept and continue with the next step.

- In this step you adjust your locale options:

![Proxmox location and timezone selection](/content/images/2025-04-13-proxmox-home-cluster-i/proxmox-location-tz.jpg)

- Here you'll set up a password for the *root* user, or administrator user of the node, and a email address for notifications:

![Proxmox admin password and notification email address](/content/images/2025-04-13-proxmox-home-cluster-i/proxmox-admin-passwd.jpg)

- Here we'll set up an IP address for the management interface. We can only setup one single network interface here, but this could be changed later:

![Proxmox management network interface configuration](/content/images/2025-04-13-proxmox-home-cluster-i/proxmox-mgmt-network-cfg.jpg)

- Before we continue, we are shown the summary of our configuration. If you agree, go ahead:

![Proxmox summary](/content/images/2025-04-13-proxmox-home-cluster-i/proxmox-summary.jpg)

- During the next two or three minutes, you'll be amused with some advertisement while the packages are installed:

![Proxmox package installation](/content/images/2025-04-13-proxmox-home-cluster-i/proxmox-pkg-install.jpg)

- If you selected the option for automatic reboot after installation, you will see this screen after reboot:

![Proxmox first boot GRUB menu](/content/images/2025-04-13-proxmox-home-cluster-i/proxmox-first-boot-grub.jpg)

- And finally, after a few seconds, you have the login screen:
![Proxmox server login screen](/content/images/2025-04-13-proxmox-home-cluster-i/proxmox-login.jpg)

## Basic setup of a node
Once you have a node correctly installed, you can access it via web, as suggested by the console prompt shown before.

- You'll get the login page, so please, log in:

![Proxmox web login](/content/images/2025-04-13-proxmox-home-cluster-i/proxmox-web-login.jpg)

- Once you log in for the first time, you'll be warned about not having an enterprise license installed.

![Proxmox license warning](/content/images/2025-04-13-proxmox-home-cluster-i/proxmox-license-warning.jpg)

- After dismissing the warning, you'll get to the main admin view:
![Proxmox main view](/content/images/2025-04-13-proxmox-home-cluster-i/proxmox-main-node-view.jpg)

- In order to add the free repositories, we'll access `pve01` (this node) -> Updates -> Repositories:

![Proxmox software repositories view](/content/images/2025-04-13-proxmox-home-cluster-i/proxmox-repositories-view.jpg)

  (and yes, we've already made some tests :smirk: )

- Here, we'll disable these two:
  - Enterprise Proxmox
  - Ceph Quincy Enterprise
- And we'll **add** these two:
  - Proxmox PVE no subscription
  - Ceph Quincy no subscription
- You have now the following options on screen:

![Proxmox software repositories view after enabling free ones](/content/images/2025-04-13-proxmox-home-cluster-i/proxmox-free-repositories-view.jpg)

- Let's configure network now in order to use the two 2.5 Gbit interfaces instead of one. We can do this through the web interface and apply changes once we agree to them. Here's what we need to do:
  - Click on `pve01` node, then System -> Network.
  - Delete current *bridge*. Click on `vmbr0` and then *Remove*
  - Click on *Create* -> *Linux bond*
    - Name: `bond0`
    - Slaves: `enp2s0 enp3s0` (the two wired network interfaces)
    - Mode: `balance-alb` . See [here](https://pve.proxmox.com/wiki/Network_Configuration#sysadmin_network_bond) for more details. This is the best configuration if you don't have a [LACP](https://en.wikipedia.org/wiki/Link_Aggregation_Control_Protocol) capable network switch.
    - Click on *Create* button.
  - Now click on *Create* -> *Linux bridge*. Fill the dialog with this:
    - Name: `vmbr0`
    - Bridge ports: `bond0` .
    - IPv4/CIDR: `192.168.18.131/24` . Same network address you used before, including network mask.
    - Gateway (IPv4): `192.168.18.1` . Same gateway as in the previous configuration.
    - Click on *Create* button.
  - The changes are previewed and you can go through them before confirming:

![Proxmox network changes preview](/content/images/2025-04-13-proxmox-home-cluster-i/proxmox-network-changes-preview.jpg)

  - If you agree, click on `Apply configuration`.

## Cluster creation
Once the three nodes are online and able to talk to each other through the network, we'll double check the [requirements](https://pve.proxmox.com/wiki/Cluster_Manager#_requirements) for creating the Proxmox cluster.

Now, for a change, we're going to create the cluster via command-line, following the steps detailed in the [docs](https://pve.proxmox.com/wiki/Cluster_Manager#pvecm_create_cluster):

1. Create the cluster.
    From the first node, we create the cluster:
    ```bash
    root@pve01:~# pvecm create tejar
    Corosync Cluster Engine Authentication key generator.
    Gathering 2048 bits for key from /dev/urandom.
    Writing corosync key to /etc/corosync/authkey.
    Writing corosync config to /etc/pve/corosync.conf
    Restart corosync and cluster filesystem
    ```
    We check the current status:
    ```bash
    root@pve01:~# pvecm status
    Cluster information
    -------------------
    Name:             tejar
    Config Version:   1
    Transport:        knet
    Secure auth:      on

    Quorum information
    ------------------
    Date:             Tue Apr 15 01:06:31 2025
    Quorum provider:  corosync_votequorum
    Nodes:            1
    Node ID:          0x00000001
    Ring ID:          1.5
    Quorate:          Yes

    Votequorum information
    ----------------------
    Expected votes:   1
    Highest expected: 1
    Total votes:      1
    Quorum:           1  
    Flags:            Quorate 

    Membership information
    ----------------------
        Nodeid      Votes Name
    0x00000001          1 192.168.18.131 (local)
    ```

2. Add second (and subsequent) node(s):
    Log in to the new node you want to join the cluster and do:
    ```bash
    root@pve02:~# pvecm add 192.168.18.131
    Please enter superuser (root) password for '192.168.18.131': ***********
    Establishing API connection with host '192.168.18.131'
    The authenticity of host '192.168.18.131' can't be established.
    X509 SHA256 key fingerprint is 80:EC:8B:A3:1F:B0:41:C6:F3:5C:E2:7A:47:6C:66:EE:12:DD:B9:EF:CE:0D:43:85:6A:70:A6:92:A7:DA:0F:27.
    Are you sure you want to continue connecting (yes/no)? yes
    Login succeeded.
    check cluster join API version
    No cluster network links passed explicitly, fallback to local node IP '192.168.18.132'
    Request addition of this node
    Join request OK, finishing setup locally
    stopping pve-cluster service
    backup old database to '/var/lib/pve-cluster/backup/config-1744672204.sql.gz'
    waiting for quorum...OK
    (re)generate node files
    generate new node certificate
    merge authorized SSH keys
    generated new node certificate, restart pveproxy and pvedaemon services
    successfully added node 'pve02' to cluster.
    ```
    In the command above we've used the IP address of an existing cluster node.

3. Repeat the previous step with the third and subsequent nodes to join the cluster.

4. Check cluster status:
    ```bash
    root@pve03:~# pvecm status
    Cluster information
    -------------------
    Name:             tejar
    Config Version:   3
    Transport:        knet
    Secure auth:      on

    Quorum information
    ------------------
    Date:             Tue Apr 15 01:14:00 2025
    Quorum provider:  corosync_votequorum
    Nodes:            3
    Node ID:          0x00000003
    Ring ID:          1.d
    Quorate:          Yes

    Votequorum information
    ----------------------
    Expected votes:   3
    Highest expected: 3
    Total votes:      3
    Quorum:           2  
    Flags:            Quorate 

    Membership information
    ----------------------
        Nodeid      Votes Name
    0x00000001          1 192.168.18.131
    0x00000002          1 192.168.18.132
    0x00000003          1 192.168.18.133 (local)
    ```

# Wrap-up
We have a working Proxmox cluster with three nodes. In upcoming posts we'll deal with [Ceph shared storage](https://pve.proxmox.com/wiki/Deploy_Hyper-Converged_Ceph_Cluster) configuration and other minor issues.

![Proxmox cluster summary view](/content/images/2025-04-13-proxmox-home-cluster-i/proxmox-cluster-summary-view.jpg)