---
layout: post
status: publish
published: true
title: ownCloud backup and rclone
author: Manuel Molina
about_author: "/us/manuel/readme.html"
author_email: mmc@pocosmhz.org
tags:
- unix
- backup
- cloud
comments: true
---
Having a \[virtual\|cloud\] drive in which I could keep some common tools, old works' backup, unfrequently used ISO images and things like that, is an issue that has been coming to my mind for so long.

As I was thinking in the order of hundreds of gigabytes, [Dropbox](https://www.dropbox.com) was no longer an option for me. I decided to go for [ownCloud](https://owncloud.com. First I tried [OwnCube](https://owncube.com/), which I have to say it's a great hosted option. However, as I have already a server in which I can host it myself, I went that direction and tried to learn something in the process.

I wanted to work in a local drive and then mirror it to that ownCloud volume, so it's available via WebDAV or even through HTTPS if needed. I don't mind accessing through a browser or even a smartphone if it's necessary. That way, I can keep my usual workflow in local and, from time to time, mirror it to the cloud.

In that step, [rsync](https://rsync.samba.org/) is the tool that will for sure come to your mind. And you're right.

Previously, when I used to sync to a simple UNIX remote folder, I did:

{% highlight bash %}
RSYNC=rsync
LOCAL_FOLDER=`echo $(cd $(dirname "$0") && pwd -P)`
REMOTE_SERVER=myserver.mydomain.com
REMOTE_FOLDER=/home/myuser/backup
$RSYNC -avz -e "ssh -C" --progress $LOCAL_FOLDER/* $REMOTE_SERVER:$REMOTE_FOLDER
{% endhighlight %}

Now, with ownCloud, you can mount a WebDAV folder locally and then sync with a similar command. However, the default behaviour for ownCloud through WebDAV is not to keep all the metadata (creation date, for instance).

I was determined to have that as well, so after a bit of research I found [rclone](https://rclone.org/) to be a perfect match. It has [options to access ownCloud](https://rclone.org/webdav/#owncloud) with the proper calls to have the needed metadata updated.

Let's give a look at how to configure that:

{% highlight bash %}
$ rclone version
rclone v1.54.0
- os/arch: linux/amd64
- go version: go1.15.7

$ rclone config
2021/03/07 19:22:50 NOTICE: Config file "/home/vagrant/.config/rclone/rclone.conf" not found - using defaults
No remotes found - make a new one
n) New remote
s) Set configuration password
q) Quit config
n/s/q> n
name> ocremote 
Type of storage to configure.
Enter a string value. Press Enter for the default ("").
Choose a number from below, or type in your own value
 1 / 1Fichier
   \ "fichier"
 2 / Alias for an existing remote
   \ "alias"
 3 / Amazon Drive
   \ "amazon cloud drive"
 4 / Amazon S3 Compliant Storage Providers including AWS, Alibaba, Ceph, Digital Ocean, Dreamhost, IBM COS, Minio, and Tencent COS
   \ "s3"
 5 / Backblaze B2
   \ "b2"
 6 / Box
   \ "box"
 7 / Cache a remote
   \ "cache"
 8 / Citrix Sharefile
   \ "sharefile"
 9 / Compress a remote
   \ "compress"
10 / Dropbox
   \ "dropbox"
11 / Encrypt/Decrypt a remote
   \ "crypt"
12 / Enterprise File Fabric
   \ "filefabric"
13 / FTP Connection
   \ "ftp"
14 / Google Cloud Storage (this is not Google Drive)
   \ "google cloud storage"
15 / Google Drive
   \ "drive"
16 / Google Photos
   \ "google photos"
17 / Hadoop distributed file system
   \ "hdfs"
18 / Hubic
   \ "hubic"
19 / In memory object storage system.
   \ "memory"
20 / Jottacloud
   \ "jottacloud"
21 / Koofr
   \ "koofr"
22 / Local Disk
   \ "local"
23 / Mail.ru Cloud
   \ "mailru"
24 / Mega
   \ "mega"
25 / Microsoft Azure Blob Storage
   \ "azureblob"
26 / Microsoft OneDrive
   \ "onedrive"
27 / OpenDrive
   \ "opendrive"
28 / OpenStack Swift (Rackspace Cloud Files, Memset Memstore, OVH)
   \ "swift"
29 / Pcloud
   \ "pcloud"
30 / Put.io
   \ "putio"
31 / QingCloud Object Storage
   \ "qingstor"
32 / SSH/SFTP Connection
   \ "sftp"
33 / Sugarsync
   \ "sugarsync"
34 / Tardigrade Decentralized Cloud Storage
   \ "tardigrade"
35 / Transparently chunk/split large files
   \ "chunker"
36 / Union merges the contents of several upstream fs
   \ "union"
37 / Webdav
   \ "webdav"
38 / Yandex Disk
   \ "yandex"
39 / Zoho
   \ "zoho"
40 / http Connection
   \ "http"
41 / premiumize.me
   \ "premiumizeme"
42 / seafile
   \ "seafile"
Storage> 37
** See help for webdav backend at: https://rclone.org/webdav/ **

URL of http host to connect to
Enter a string value. Press Enter for the default ("").
Choose a number from below, or type in your own value
 1 / Connect to example.com
   \ "https://example.com"
url> https://oc.mydomain.com/remote.php/webdav/     
Name of the Webdav site/service/software you are using
Enter a string value. Press Enter for the default ("").
Choose a number from below, or type in your own value
 1 / Nextcloud
   \ "nextcloud"
 2 / Owncloud
   \ "owncloud"
 3 / Sharepoint
   \ "sharepoint"
 4 / Other site/service or software
   \ "other"
vendor> 2
User name
Enter a string value. Press Enter for the default ("").
user> manuelmc
Password.
y) Yes type in my own password
g) Generate random password
n) No leave this optional password blank (default)
y/g/n> y
Enter the password:
password:
Confirm the password:
password:
Bearer token instead of user/pass (e.g. a Macaroon)
Enter a string value. Press Enter for the default ("").
bearer_token> 
Edit advanced config? (y/n)
y) Yes
n) No (default)
y/n> 
Remote config
--------------------
[ocremote]
type = webdav
url = https://oc.mydomain.com/remote.php/webdav/
vendor = owncloud
user = manuelmc
pass = *** ENCRYPTED ***
--------------------
y) Yes this is OK (default)
e) Edit this remote
d) Delete this remote
y/e/d> y
Current remotes:

Name                 Type
====                 ====
ocremote             webdav

e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q> q
{% endhighlight %}

Now you can check that it works by doing:

{% highlight bash %}
$ rclone ls ocremote:
  6662575 ownCloud Manual.pdf
   538942 Photos/Lake-Constance.jpg
   243733 Photos/Portugal.jpg
   228789 Photos/Teotihuacan.jpg
    36227 Documents/Example.odt
{% endhighlight %}

Now, replacing the previous rsync/SSH example I cited above, you can do this to achieve the same result with a WebDAV destination:

{% highlight bash %}
RCLONE=rclone
LOCAL_FOLDER=`echo $(cd $(dirname "$0") &amp;&amp; pwd -P)`
REMOTE_FOLDER=ocremote:backup
$RCLONE sync -P $LOCAL_FOLDER $REMOTE_FOLDER --progress --multi-thread-streams=5
{% endhighlight %}
