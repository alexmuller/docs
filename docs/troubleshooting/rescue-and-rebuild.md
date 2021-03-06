---
author:
  name: Linode
  email: docs@linode.com
description: 'Our guide to rescuing and rebuilding your Linode.'
keywords: 'rescue,rebuild'
license: '[CC BY-ND 3.0](http://creativecommons.org/licenses/by-nd/3.0/us/)'
alias: ['rescue-and-rebuild/','troubleshooting/finnix-rescue-mode/']
modified: Thursday, June 19th, 2014
modified_by:
  name: Linode
published: 'Thursday, May 31st, 2012'
title: Rescue and Rebuild
---

Even the best system administrators have to deal with accidents and unplanned events. Fortunately, the Linode Manager has a number of tools to assist you in the unlikely event that catastrophe strikes your VPS. This guide shows you how to use the tools at your disposal. You can boot your Linode into *rescue mode* to perform system recovery tasks and transfer data off your disk images, if necessary. And if all else fails, you can *rebuild* your Linode from a backup or start over with a fresh Linux distribution.

Rescuing
--------

If you suspect that your primary filsystem is corrupted, use the Linode Manager to boot your Linode into rescue mode. This is a safe environment for performing many system recovery and disk management tasks. Rescue mode is based on the [Finnix recovery distribution](http://www.finnix.org/), a self-contained and bootable Linux distribution. You can also use rescue mode for tasks other than disaster recovery, such as formatting disk images to use different filesystems, copying data between disk images, and downloading files from a disk image via SSH and SFTP.

### Booting into Rescue Mode

Here's how to boot your Linode into rescue mode:

1.  Log in to the [Linode Manager](https://manager.linode.com).
2.  Click the **Linodes** tab. A list of your virtual private servers appears.
3.  Select a Linode. The Linode's dashboard appears.
4.  Click the **Rescue** tab. The webpage shown below appears.

[![The Rescue tab in the Linode Manager](/docs/assets/1000-rescue3-1.png)](/docs/assets/1000-rescue3-1.png)

5.  In the **Rescue Mode** section, select the disk images you want to be mounted.

 {: .note }
>
> Make a note of where the disk images are located. For example, in the screenshot shown above, the Ubuntu disk image is at `/dev/xvda`. You will need this information later.

6.  Click **Reboot into Rescue Mode**. The Linode's dashboard appears. Watch the *Host Job Queue* on the Dashboard to monitor the progress.

Your Linode has been booted into rescue mode. To access it, follow the instructions in the next section.

### Connecting to a Linode Running in Rescue Mode

To access your Linode when it's running in rescue mode, you'll need to use the Linode shell (Lish). For instructions, see [Console Access](/docs/remote-access#sph_console-access) and [Using the Linode Shell (Lish)](/docs/troubleshooting/using-lish-the-linode-shell). Once you've successfully logged in, you'll see the window shown below.

[![Lish console](/docs/assets/1002-rescue1-2.png)](/docs/assets/1002-rescue1-2.png)

Now you've connected to your Linode, you can start doing stuff in rescue mode.

### Performing a File System Check

You can use the `fsck` system utility (short for "file system check") to check the consistency of file systems and repair any damage detected. If you suspect that your Linode's file system is corrupted, you should run `fsck` to check for and repair any damage. Here's how:

1.  Enter the `df` command to verify that your primary disk images are not currently mounted. Your primary disk images should not appear in the list. For example, when [we booted into rescue mode earlier](#booting-into-rescue-mode), we specified two disk images: Ubuntu at `/dev/xvda` and Swap at `/dev/xvdb`, neither of which are shown as being mounted in the screenshot below.

[![Output of df command](/docs/assets/999-rescue2.png)](/docs/assets/999-rescue2.png)

 {: .note }
>
> You should never run `fsck` on a mounted disk. Do not continue unless you're sure that the target disk image is unmounted.

2.  To verify the location of your disk images, enter the `fdisk -l` command. The disk image layout will appear, as shown below. Notice that the Ubuntu disk image is `/dev/xvda`, the Swap disk image is `/dev/xvdb`, and the Finnix partition is `/dev/xvdh`.

[![Output of fdisk -l command](/docs/assets/1001-rescue4.png)](/docs/assets/1001-rescue4.png)

3.  Run `fsck` by entering the following command, replacing `/dev/xvda` with the location of the disk image you want to check and repair:

        e2fsck -f /dev/xvda

4.  If no problems are detected, `fsck` will display a message indicating that the file system is "clean," as shown below.

[![Output of e2fsck command](/docs/assets/1003-rescue5.png)](/docs/assets/1003-rescue5.png)

5.  If `fsck` determines that there is a problem with your file system, it will perform several tests and prompt you to fix problems as they are found, as shown below. Press enter to automatically attempt to fix the problems.

[![Output of e2fsck command](/docs/assets/1007-rescue6-1.png)](/docs/assets/1007-rescue6-1.png)

Once the file system check completes, any problems detected should be fixed. You can try restarting the Linode now. With any luck, `fsck` fixed your problem and the Linode will boot normally.

### Mounting Disks

By default, your disk images are not mounted when your Linode boots into rescue mode. However, you can manually mount a disk while your Linode is running in rescue mode to perform system recovery and maintenance tasks. Enter the following command to mount a disk in rescue mode, replacing `/dev/xvda` with the location of the disk image you want to mount:

    mount -o barrier=0 /dev/xvda

Disks that contain a single file system will have mount points under `/media` in the rescue environment's `/etc/fstab` file. To view the directories on the disk, enter the following command: :

    ls /media/xvda

Now you can read and write to files on the mounted disk.

### Change Root

Change root is the process of changing your working root directory. When you change root (chroot) to your Linode root disk image, you will be able to run commands as though you are logged into that system.

Chroot will allow you to change user passwords, remove/install packages, and do other system maintanance and recovery tasks.

Before you can use chroot, you need to mount your root disk image with execute permissions:

    mount -o exec,barrier=0 /dev/xvda

Then to create the chroot, you need to mount the temporary filesystems:

    cd /media/xvda
    mount -t proc proc proc/
    mount -t sysfs sys sys/
    mount -o bind /dev dev/
    mount -t devpts pts dev/pts/

Chroot to your disk image with the following command:

    chroot /media/xvda /bin/bash

To exit the chroot and get back to Finnix type "exit" :

    exit

### Starting SSH

The Finnix recovery distribution does not automatically start an SSH server, but you can start one manually. This is useful if your Linode won't boot and you need to copy files off of the disks. You can also copy entire disk images over SSH. Here's how to start SSH:

1.  Set the `root` password by entering the following command:

        passwd

2.  Enter the password for the `root` user.
3.  Start the SSH server by entering the following command:

        /etc/init.d/ssh start

Now you can access mounted disks with an SFTP client by using the `root` user and the password you just set. For instructions on connecting with an SFTP client, see the [File Transfer reference manuals](/docs/networking/file-transfer). For instructions on copying an entire disk image over SSH, see [Copy a Disk Image Over SSH](/docs/linode-platform/migration/copy-disk-image-over-ssh).

### Installing Packages

The Finnix recovery distribution is based on Debian, so you can use the `apt` package management system to install additional software packages in the temporary rescue environment. For example, you could install and run the `htop` utility by issuing the following commands:

    apt-get update
    apt-get install htop
    htop

The software packages you install will be available as long as your Linode is running in rescue mode.

Rebuilding
----------

If you can't rescue an existing disk image, it's time to rebuild your Linode. There are a couple different ways you can do this. You restore from an existing backup and return your Linode to a previous state. If you don't have backups, you can copy files off an existing disk image, erase everything, and start over again from scratch.

### Restoring from Backup

If you previously enabled the Linode Backup Service, you may be able to restore one of the backups to your Linode. For instructions on restoring from a backup created by the Linode Backup Service, see [Restoring from a Backup](/docs/backup-service#sph_id2). If you created backups with an application other than the Linode Backup Service, review the application's instructions to restore a backup to your Linode.

### Recovering From a System Compromise

Did an unauthorized intruder gain access to your Linode? Since it is virtually impossible to determine the full scope of an attacker's reach into a compromised system, you should never continue using a compromised Linode. We recommend that you follow the instructions in [Recovering from a System Compromise](/docs/troubleshooting/compromise-recovery). You'll need to create a new Linode, copy your existing data from the old Linode to the new one, and then swap IP addresses.

### Erasing Everything and Starting Over

Sometimes it's just easier to erase all of your Linode's disk images and start over with a fresh Linux distribution. This is the "nuclear option" that will erase *everything* on your Linode.

 {: .note }
>
> If you'd like to deploy a new Linux distribution without erasing your existing disk images, see [Creating a Disk Image with a Linux Distribution Installed](/docs/disk-images-config-profiles#sph_creating-a-disk-image-with-a-linux-distribution-installed). This is a better option for those who need create a new distribution, but also need to save their existing data.

Here's how to rebuild your Linode from scratch:

1.  If you need to copy files from an existing disk image, [start SSH](#starting-ssh) and then use an SFTP client to copy files to your computer.
2.  Log in to the [Linode Manager](https://manager.linode.com).
3.  Click the **Linodes** tab. A list of your virtual private servers appears.
4.  Select a Linode. The Linode's dashboard appears.
5.  Click the **Rebuild** tab. The webpage shown below appears.

[![The "Rebuild" tab of the Linode Manager](/docs/assets/1006-rescue7-small.png)](/docs/assets/1005-rescue7.png)

6.  Select Linux distribution from the **Distribution** menu.
7.  Enter a size for the new disk image in the **Deployment Disk Size**.
8.  From the **Swap Disk** disk menu, select a size for the disk image. We strongly recommend using the default size.
9.  Enter a root password for your Linode in the **Root Password** field.
10. Click **Rebuild**. The Linode's dashboard appears. The Linode Manager deletes your existing disk images and installs the Linux distribution you selected in a new disk image.
11. Click **Boot** to turn on the Linode.

Your Linode will boot with the newly-installed Linux distribution. See the [Getting Started guide](/docs/getting-started) to reprovision your server.



