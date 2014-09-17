---
layout: post.html
title: Cloning an Ubuntu AWS EC2 Instance Locally (Hyper V)
tags: [aws, hyper-v, ubuntu]
---

## Overview

Basically we are going make an image of the EBS volumes attached to the instance you want to run locally and, download them, and finally extract them onto .VHDX  and spin up an new hyper-v machine using that .VHDX

## Create the image of your EBS volume

### First we need to create a micro linux instance to aid in the creation of the image.

1. From your EC2console  click the "Launch Instance button"
2. From here we can select the "Classic Wizard" and click "Continue"
3. We'll use the Amazon Linux AMI in 64bit
4. Now select a "T1 Micro" instance type and place it in the Availability Zone that the instance we are downloading is currently located.
5. Go ahead and continue through and name your instance "Migration Helper"
6. Use the existing key that the instance we are moving currently uses
7. Add it to the same security group(s) as well

### Next we'll create new EBS volumes, an empty one and a clone of the EBS volume from the instance.

1. Find the  EBS ID of your volume by selecting it in the console and scrolling down in the information panel and clicking on the device in "Block Devices"
2. Move to the "Volumes" console and find your EBS volume.
3. Right click the volume and select "Create snapshot"
4. Name the snapshot and set it to create.
5. While the snapshot is being created go ahead and create an empty EBS volume the same size as the volume you are currently snapshotting in the same availability zone as the instance we are moving.
6. Attach the blank volume to your "Migration Helper" instance as /dev/sdf.
7. Once the snapshot has been created move over to the snapshots console and find the snapshot.
8. Right click the snapshot and select "Create Volume from Snapshot"
9. Size it the same as the empty volume you just created and add it to the same availability zone as well.
10. Once the volume has been created attach it you your "Migration Helper" instance as /dev/sdg

### Finally we will create the image

+ SSH into your "Migration Helper instance"
+ Switch to the root user
```bash
sudo su -

+ Create a file system on the blank volume
```bash
mkfs.ext3 /dev/sdf

+ Create a mount point and mount the blank volume

```bash
    mkdir /mnt/img
    mount /dev/sdf /mnt/img
```

+ Using dd create an image of the cloned volume.
```bash
dd if=/dev/sdg | gzip -c  > /mnt/img/sda1.img.gz
```

+ Once the image has been created we need to change ownership of the /mnt/img directory
```bash
chown -R ec2-user:ec2-user /mnt/img
```

## Downloading the image

### First we need to add the AWS key to your local Hyper-V instance

+ Create a directory called key in your home directory
```bash
mkdir ~/key
```
+ Create a file called aws.pem
```bash
vim aws.pem
```
+ Paste your private key information into the file, then save and close it.
+ Change permissions so only the owner can read and write to the key file.
```bash
chmod 600 aws.pem
```

### Now we're ready to download the image.

+ Create a directory in your home directory to store the image.
```bash
mkdir ~/imgStore
```
+ Now using scp we will copy down the image file we created from our "Migration Helper"
```bash
scp -i ~/key/aws.pem ec2-user@us-west-ip:/mnt/img/sda1.img.gz ~/imgStore
```
## Create a VHDX for the cloned machine

### First we need to create a blank VHDX to unpack the cloned image onto

+ Open Hyper-V Manager
+ Select New>Hard Disk...
+ Choose a Fixed size disk
+ Choose a name and location for the blank drive
+ Size it appropriately to fit the unpacked image
+ Review your selections and click finish

### Now we need to attach the blank .VHDX to our local Hyoer-V linux instance

+ Shut down the instance
+ Open the settings forthe instance
+ Select IDE Controller 0
+ Add a hard drive
+ Enter the path of your newly created blank .VHDX
+ Click OK, and start the machine back up

### Before we can unpack the image onto the new drive we have to re-create the partition table

+ Start parted
```bash
parted
```
+ Set device to the newly attached blank drive
```bash
(parted) select /dev/sdb
```
+ Create the the partition that you will be cloning the image we brought down from EC2 onto. (substitute your own start end values)
```bash
(parted) mkpart primary ext4 0 30000
```
+ Next create your swap partition (substitute your own start end values)
```bash
(parted) mkpart primary linux-swap 30000 32048
```

+ Finally we will set the first partition to boot
```bash
(parted) set 1 boot
```

+  Exit parted
```bash
exit
```
### Now we unpack the image

As root
```bash
gunzip -c ~/imgStore/sda1.img.gz | dd of=/dev/sdb1
```
## Final config and boot

+ Because of how EC2 handles virtual instances there are a few things that we need to make a few more changes to get this VM to boot in a timely fashion.

+ First let's mount the new drive and put ourselves in a chroot shell

```bash
    mount /dev/sdb1 /mnt
    mount --bind /dev /mnt/dev
```

+ Now we can chroot into the system and mount some important virtual filesystems

```bash
    chroot /mnt
    mount -t proc none /proc
    mount -t sysfs none /sys
    mount -t devpts none /dev/pts
    export HOME=/root
    export LC_ALL=C
```

Because EC2 instances don't actually have a kernel installed and instead have it loaded by the Xen server on boot, we need to install a kernel via apt-get.

Before we can do this though we need to fix our hosts file so we can access the internet.

We'll do this by copying the host file currently  being used by our current boot drive.

+ In a new terminal issue these commands

```bash
    sudo cp /mnt/etc/hosts /mnt/etc/hosts.old
    sudo cp /etc/hosts /mnt/etc/hosts
    sudo cp /etc/resolv.conf /mnt/etc/resolv.conf
    exit
```

### Onto installing the kernel!

There are three kernels that you can install, and it's important to choose the correct one.

Ubuntu 12.10 and above
```bash
apt-get install -y linux-image-generic
```
Ubuntu 12.04 64-bit
```bash
apt-get install -y linux-image-generic
```
Ubuntu 12.04 32-bit < 3gig of RAM
```bash
apt-get install -y linux-image-generic
```
Ubuntu 12.04 32-bit >3 gig of RAM
```bash
apt-get install -y linux-image-generic-pae
```
Ubuntu 11.10 and below Server
```bash
apt-get install -y linux-image-server
```
Ubuntu 11.10 and below 64-bit
```bash
apt-get install -y linux-image-generic
```
Ubuntu 11.10 and below 32-bit < 3gig of RAM
```bash
apt-get install -y linux-image-generic
```
Ubuntu 11.10 and below 32-bit >3 gig of RAM
```bash
apt-get install -y linux-image-generic-pae
```

Once the kernel is installed we can go ahead and remove the cloud-init package to improve boot times
```bash
apt-get remove cloud-init
```
## Boot the virtual machine

Now we create a new virtual machine in Hyper-V using our newly cloned disk and boot.
