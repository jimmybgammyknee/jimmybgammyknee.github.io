---
layout: post
title: "Australian Academic Data Sharing from the terminal"
date:   2017-07-24
---

**Author Preface:** Yes I know its been a while between blog posts! Just a little note to say that this site will be changing a little soon from a personal blog to a group blog. I want to promote the work of my awesome team at the SAHMRI Bioinformatics Core a bit more, so keep a look at for updates

---

I commonly run analyses for people from other institutions and I always run into issues regarding sending data. Often I will use a [Dropbox](https://www.dropbox.com) or [Box](http://www.box.com)-style sharing tools, which can be problematic when sending really big files. They also don't work well with command-line infrastructure either, and are more targeted for people who want a folder constantly synced or FTP/SFTP clients.

I've known about [CloudStor](https://www.aarnet.edu.au/network-and-services/cloud-services-applications/cloudstor) for a long period of time and used it extensively in my previous positions. It gives you access to 1TB free storage for Academic institutions and doesn't have file size restrictions like other services do. Ultimately I really didn't like the web-interface that much and therefore drifted off towards other services over the past few years.

However I recently had a project in which I had to send a >30GB file securely and thought I would give it another crack. But thankfully I found a [nice little tutorial](https://support.aarnet.edu.au/hc/en-us/articles/115007168507-Can-I-use-the-command-line-or-WebDav-) on using [WebDAV](https://en.wikipedia.org/wiki/WebDAV) and command-line tools to upload data from our local server instead of having to transfer the file from my computer and _then_ to the CloudStor shared directory.

Here I have just used the tool [`rclone`](https://rclone.org/) but you can easily use some other command-line tools with WebDAV support implemented.

## 1. Download and Install `rclone`

This is the easy part. If you're using `anaconda` or `miniconda` or `minimamba` package management systems, you can just run the following (im using `mamba` here):

```
(base) [Jimmy@Server ~]$ mamba install -c hcc rclone

              __    __    __    __
             /  \  /  \  /  \  /  \
            /    \/    \/    \/    \
███████████████/  /██/  /██/  /██/  /████████████████████████
          /  / \   / \   / \   / \  \____
         /  /   \_/   \_/   \_/   \    o \__,
        / _/                       \_____/  `
        |/
    ███╗   ███╗ █████╗ ███╗   ███╗██████╗  █████╗
    ████╗ ████║██╔══██╗████╗ ████║██╔══██╗██╔══██╗
    ██╔████╔██║███████║██╔████╔██║██████╔╝███████║
    ██║╚██╔╝██║██╔══██║██║╚██╔╝██║██╔══██╗██╔══██║
    ██║ ╚═╝ ██║██║  ██║██║ ╚═╝ ██║██████╔╝██║  ██║
    ╚═╝     ╚═╝╚═╝  ╚═╝╚═╝     ╚═╝╚═════╝ ╚═╝  ╚═╝

    Supported by @QuantStack

    GitHub:  https://github.com/QuantStack/mamba
    Twitter: https://twitter.com/QuantStack

█████████████████████████████████████████████████████████████

Getting  https://conda.anaconda.org/hcc/linux-64
Getting  https://conda.anaconda.org/hcc/noarch
Getting  https://repo.anaconda.com/pkgs/main/linux-64
Getting  https://repo.anaconda.com/pkgs/main/noarch
Getting  https://repo.anaconda.com/pkgs/free/linux-64
Getting  https://repo.anaconda.com/pkgs/free/noarch
Getting  https://repo.anaconda.com/pkgs/r/noarch
Getting  https://repo.anaconda.com/pkgs/r/linux-64

Looking for: ['rclone']

13330 packages in https://repo.anaconda.com/pkgs/free/linux-64
5257 packages in https://repo.anaconda.com/pkgs/r/linux-64
13608 packages in https://repo.anaconda.com/pkgs/main/linux-64
29 packages in https://repo.anaconda.com/pkgs/free/noarch
212 packages in https://repo.anaconda.com/pkgs/r/noarch
649 packages in https://repo.anaconda.com/pkgs/main/noarch
15 packages in https://conda.anaconda.org/hcc/noarch
532 packages in https://conda.anaconda.org/hcc/linux-64

## Package Plan ##

environment location: /XXXXX/XXXXX/XXXXX/minimamba

added / updated specs:
- rclone


The following packages will be downloaded:

package                    |            build
---------------------------|-----------------
rclone-1.44                |                0         8.6 MB  hcc
------------------------------------------------------------
                                       Total:         8.6 MB

The following NEW packages will be INSTALLED:

rclone             hcc/linux-64::rclone-1.44-0


Proceed ([y]/n)? y


Downloading and Extracting Packages
rclone-1.44          | 8.6 MB    | ################################################################################################################### | 100%
Preparing transaction: done
Verifying transaction: done
Executing transaction: done
```

Easy as! That command should work on `conda` as well

Note: If you dont have a package manager then there are [installation instructions on the rclone site](https://rclone.org/install/). If you're on MacOSX, I would install this using `homebrew`

```
brew install rclone
```

## 2. It's a setup!

Next we need to tell `rclone` our credentials. The commands that I actually ran are in **bold** (just like the tutorial btw).

**Note: Before you do this, I would logon to your CloudStor account and login with your institutes SSO credentials (basically your University of Academic username and password). Go to the "Settings" Icon at the top right of the page and go to the section titled "Sync Password". Create a password here so you can use it below.**

```
(base) [Jimmy@Server ~]$ rclone config
2019/07/24 09:20:11 NOTICE: Config file "/XXXXX/XXXXX/.config/rclone/rclone.conf" not found - using defaults
No remotes found - make a new one
n) New remote
s) Set configuration password
q) Quit config
n/s/q> **n**
name> **CloudStor**
Type of storage to configure.
Enter a string value. Press Enter for the default ("").
Choose a number from below, or type in your own value
 1 / A stackable unification remote, which can appear to merge the contents of several remotes
   \ "union"
 2 / Alias for a existing remote
   \ "alias"
 3 / Amazon Drive
   \ "amazon cloud drive"
 4 / Amazon S3 Compliant Storage Providers (AWS, Ceph, Dreamhost, IBM COS, Minio)
   \ "s3"
 5 / Backblaze B2
   \ "b2"
 6 / Box
   \ "box"
 7 / Cache a remote
   \ "cache"
 8 / Dropbox
   \ "dropbox"
 9 / Encrypt/Decrypt a remote
   \ "crypt"
10 / FTP Connection
   \ "ftp"
11 / Google Cloud Storage (this is not Google Drive)
   \ "google cloud storage"
12 / Google Drive
   \ "drive"
13 / Hubic
   \ "hubic"
14 / JottaCloud
   \ "jottacloud"
15 / Local Disk
   \ "local"
16 / Mega
   \ "mega"
17 / Microsoft Azure Blob Storage
   \ "azureblob"
18 / Microsoft OneDrive
   \ "onedrive"
19 / OpenDrive
   \ "opendrive"
20 / Openstack Swift (Rackspace Cloud Files, Memset Memstore, OVH)
   \ "swift"
21 / Pcloud
   \ "pcloud"
22 / QingCloud Object Storage
   \ "qingstor"
23 / SSH/SFTP Connection
   \ "sftp"
24 / Webdav
   \ "webdav"
25 / Yandex Disk
   \ "yandex"
26 / http Connection
   \ "http"
Storage> **24**
** See help for webdav backend at: https://rclone.org/webdav/ **

URL of http host to connect to
Enter a string value. Press Enter for the default ("").
Choose a number from below, or type in your own value
 1 / Connect to example.com
   \ "https://example.com"
url> **https://cloudstor.aarnet.edu.au/plus/remote.php/webdav/**
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
vendor> **2**
User name
Enter a string value. Press Enter for the default ("").
user> **jimmy@jimmy.has.an.academic.email.com**
Password.
y) Yes type in my own password
g) Generate random password
n) No leave this optional password blank
y/g/n> **y**
Enter the password:
password: **[MY APP PASSWORD]**
Confirm the password:
password: **[MY APP PASSWORD AGAIN]**
Bearer token instead of user/pass (eg a Macaroon)
Enter a string value. Press Enter for the default ("").
bearer_token> **[I JUST PRESSED ENTER HERE]**
Remote config
--------------------
[CloudStor]
type = webdav
url = https://cloudstor.aarnet.edu.au/plus/remote.php/webdav/
vendor = owncloud
user = jimmy@jimmy.has.an.academic.email.com
pass = *** ENCRYPTED ***
--------------------
y) Yes this is OK
e) Edit this remote
d) Delete this remote
y/e/d> **y**
Current remotes:

Name                 Type
====                 ====
CloudStor            webdav

e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q> **q**
```

You're all setup!

## 3. Usage

Now we get to use it! For uploading data I ran the following command:

```
rclone copy --progress --transfers 8 my_file CloudStor:my_file
```

This will use 8 parallel connection streams to transfer data between my server and my CloudStor account.

For Downloading onto the server, you can run:

```
rclone copy --progress --transfers 8 CloudStor:my_file ~/
```

This will put the file in my home directory (`~/`) but obviously you can put it anywhere.

---

And thats it! You now have the ability to transfer straight to a cloud directory from your work server without the pain of download/mounting and dragging files onto screens etc.

Many thanks to the AARnet website for a lot of this information and happy transferring!
