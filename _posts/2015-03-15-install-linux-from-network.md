---
layout: post
title:  "Install Linux from Network"
date:   2015-03-15 23:30:00
tags:
  - "#pxe "
  - "#dhcp "
  - "#tftp "
  - "#os "
image: "../posts-images/install-linux-from-network.jpg"
---

Installing a new OS from scratch (physical or virtual) sucks.

From floppy drives to virtual clones, the process has always involved manual instructions and cold coffee. In the Linux scenario, PXE boot can help a little bit.

Basically, PXE is a protocol that allows machines to get something from network booting, usually an image.
What it means for OS deployment is that it can pass a kernel image to the machine over the network with arguments, such as local repositories and configuration files, allowing a full remote and automated installation.

This video contains a good explanation.

<iframe align="center" width="420" height="315" src="http://www.youtube.com/embed/zpzPuK6LNQ4" frameborder="0" allowfullscreen> </iframe>
<br />


You're gonna need a DHCP, TFTP, HTTP and NFS server.

* The DHCP tells the machine where to find the boot image;
* The TFTP hosts the boot, kernel and initrd images. It has a menu file with all the arguments for the OS installation;
* The NFS has the CentOS installation files (extracted ISO);
* The HTTP has the Ubuntu files (extracted ISO) and a the seed files (this files contains the installation arguments, like timezone, disk layout and so on).

### DHCP ###
In the DHCP server, you must setup two options:

* next-server with the TFTP server IP;
* filename with the PXE boot image.

{% highlight bash %}
next-server <tftp_ip>;
filename "pxelinux.0";
{% endhighlight %}

### TFTP ###
The TFTP server hosts some files:

  * pxelinux.0: the binary boot image;
  * kernel/initrd: kernel/initial ramdisk images, downloaded to the machine over the network;
  * menu.c32: the binary menu used to choose the OS to install;
  * pxelinux.cfg/default: displays a menu with the location of the kernel images and arguments. Like a grub file;
{% highlight bash %}
# best menu, no prompt for boot: and timeout 10 sec
# the .c32 files and the pxelinux.0 are from syslinux package
DEFAULT menu.c32
PROMPT 0
TIMEOUT 100  

MENU TITLE OS Deploy Boot Menu
MENU AUTOBOOT Starting Local System in # seconds

LABEL localboot
  MENU LABEL Boot from first hard drive
  # use chain.c32 instead of LOCALBOOL .
  # Win8.1 crashed after exited PXE boot
  COM32 chain.c32
  APPEND hd0

# Uses KS file from HTTP and mirror from NFS
LABEL CentOS-7.0-1406-x86_64-Minimal
  MENU LABEL CentOS-7.0-1406-x86_64-Minimal
  kernel ../CentOS-7.0-1406-x86_64-Minimal/vmlinuz
  append initrd=../CentOS-7.0-1406-x86_64-Minimal/initrd.img ks=http://<http_server>/ks_seed/ks-centos7-minimum.cfg

# Uses Seed file from HTTP and mirror from HTTP
LABEL ubuntu-14.04-netboot
  MENU LABEL ubuntu-14.04-netboot
  kernel ../ubuntu-14.04-netboot/linux
  append initrd=../ubuntu-14.04-netboot/initrd.gz hostname=localhost localdomain=localdomain ip=dhcp auto=true url=http://<http_server>/ks_seed/ubuntu-14.04-netboot.seed

{% endhighlight %}

This menu generates this boot screen:
![PXE Boot Screen](../../../posts-images/pxe.png)

### NFS ###
The NFS server exports the installation files from CentOS. It is actually the ISO extracted.

### HTTP ###
The webserver hosts the seeds and Ubuntu installation files (again, the ISO extracted).

### Seed Files ###
Instead of manually typing the parameters during the installation process, the seed files allow you to pre configure it.
CentOS/RHEL has different names and syntax. The firts one's called Kickstart File (KS) and the second Preseed file. Here I am going to post what I'm using in my lab, but the official documentation in the references has all the options available.

**CentOS/RHEL KS Seed**
{% highlight bash %}
# Automated install for CentOS 
# No need for comments, simple conf

install
nfs --server=<nfs-server> --dir=/data/images/CentOS-7.0-1406-x86_64-Minimal
network --bootproto=dhcp
selinux --disabled
firewall --disabled

eula --agreed
text

lang en_US.UTF-8
keyboard us
timezone America/Sao_Paulo

services --enabled=sshd
skipx

bootloader --location=mbr
zerombr
clearpart --all --initlabel

# Use sda, if using KVM change do vda
ignoredisk --only-use=sda

# Standard Disk
#part swap --asprimary --fstype="swap" --size=1024
#part swap --recommended
#part /home --fstype ext4 --size=200
#part / --fstype ext4 --size 1 --grow

# boot must be out of LVM
part /boot --fstype ext4 --size=200                                                                 

# LVM
part pv.01 --grow --size 1
volgroup vg_main pv.01
logvol / --fstype=ext4 --name=lv_root --vgname=vg_main --grow --size=8192 --maxsize=8192
logvol swap --name=lv_swap --vgname=vg_main --recommended

auth --useshadow --enablemd5
rootpw --iscrypted $1$/51P1z6E$eigfbFhKGOM2UZBTOpUCh/

reboot

%packages
@core
%end
{% endhighlight %}

**Ubuntu Preseed** 
{% highlight bash %}
d-i debian-installer/locale string en_US

d-i console-setup/ask_detect boolean false
d-i keyboard-configuration/layoutcode string us

d-i netcfg/choose_interface select auto
d-i netcfg/dhcp_timeout string 60
d-i netcfg/get_hostname string localhost
d-i netcfg/get_domain string localdomain
d-i netcfg/dhcp_hostname string localhost

# "Install the system, step failed" bug
d-i live-installer/net-image string http://<http-server>/ubuntu14.04/install/filesystem.squashfs

# Local mirror via HTTP
d-i mirror/country string manual
d-i mirror/http/hostname string <http-server> 
d-i mirror/http/directory string /ubuntu14.04
d-i mirror/http/proxy string

d-i clock-setup/utc boolean true
d-i time/zone string America/Sao_Paulo

d-i partman-auto-lvm/guided_size string max
d-i partman-auto/choose_recipe select home
d-i partman-auto/method string lvm
d-i partman-lvm/confirm boolean true
d-i partman-lvm/confirm_nooverwrite boolean true
d-i partman-lvm/device_remove_lvm boolean true
d-i partman-md/confirm boolean true
d-i partman-md/confirm_nooverwrite boolean true
d-i partman-partitioning/confirm_write_new_label boolean true
d-i partman/choose_partition select finish
d-i partman/confirm boolean true
d-i partman/confirm_nooverwrite boolean true
d-i partman/default_filesystem string ext4
d-i partman/mount_style select uuid
d-i partman/unmount_active boolean true

d-i passwd/root-login boolean true
d-i passwd/root-password-crypted password $1$/51P1z6E$eigfbFhKGOM2UZBTOpUCh/

d-i passwd/user-fullname string lab
d-i passwd/username string lab
d-i passwd/user-password-crypted password $1$/51P1z6E$eigfbFhKGOM2UZBTOpUCh/
d-i user-setup/allow-password-weak boolean true
d-i user-setup/encrypt-home boolean false

d-i pkgsel/update-policy select none
d-i pkgsel/upgrade select none

# Select the server role
# none = minimum
d-i tasksel/first multiselect none 

d-i grub-installer/only_debian boolean true
d-i grub-installer/with_other_os boolean true

d-i preseed/late_command string in-target /bin/hostname localhost ;

d-i finish-install/reboot_in_progress note
{% endhighlight %}

### All together now! ###
How does it all work together?

1. The machine makes a network boot.
2. The DHCP server delivers an IP address, a TFTP address and the file to boot.
3. The machine locates the TFTP server and downloads the boot image with a menu file.
4. The menu is displayed and the user chooses one option.
5. By the option, the kernel/initrd images are downloaded with the specific parameters.
6. The kernel loads the seed file, which shows the installation options and where the files are.
7. The installation begins, download the files, prepare the OS and finishes.
8. Machine reboots. The OS is ready to be used.

### Troubleshooting ###
Even though it's cool, this proccess has too many parts that may fail. If it's not working, here's a list of things to look out:

* Machine boot order;
* IP address/names of every server;
* Firewall rules (TFTP, HTTP, NFS);
* TFTP configuration and access;
* DHCP configuration and IP lease;
* NFS exports;
* HTTP configuration and access;
* Location of images, source files;
* Menu parameters;
* Seeds parameters.

### Files and References ###
These files are on my GitHub (they may be slightly different from the post):

{% highlight bash %}
git clone https://github.com/jonatasbaldin/pxeboot.git
{% endhighlight %}

The seed files have a lot of parameters. You can read: 

* [RHEL 7 Kickstart File Reference](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Installation_Guide/sect-kickstart-syntax.html)
* [Ubuntu Preseed Reference](https://help.ubuntu.com/10.04/installation-guide/i386/preseed-using.html)
