---
layout: post
title: ISCSI connection inexplicably going offline using Ubuntu 18.04 & Windows Server 2019
date: 2020-09-21 13:56
category: linux
author: Zeb Rasco
tags: [open-iscsi, iscsi ]
summary: 
---

A few weeks ago I upgraded my file server from Windows 2012 R2 to Windows 2019. This file server hosts several iSCSI targets, one of which is a 1.5TB ext4 volume for use with [NextCloud](https://www.nextcloud.com), an open-source file hosting platform that you can run on your own hardware. I use it to sync my files across multiple computers.

Unfortunately, I discovered the NextCloud server was unreliable after the file server upgrade. NextCloud would work for awhile, and then the site would either randomly return a 500 internal server error, or, would complain about the data directory being un-writable.   

This took me about a day to figure out, but once I found the fix, it was simple. [Click here to just skip to the fix!](#the-fix)

### My NextCloud setup

I have a general-purpose LAMP server set up in my lab, running Ubuntu 18.04.2 (and Linux kernel 4.15), and hosting the NextCloud service. It has a 100GB virtual hard drive provisioned for the OS. However, I needed more storage for NextCloud, and since I already had a file server set up with an iSCSI target service, I decided to simply create a new iSCSI target and point NextCloud to it.

So I set up an iSCSI target on the file server (then running Windows 2012 R2), installed open-iscsi on the Ubuntu server, ran a few commands using iscsiadm, changed the target config to autoconnect, and set up an /etc/fstab entry to mount the iSCSI disk on bootup. Then I pointed NextCloud to the mount directory, ran the initialization via the NextCloud web interface, and viola. Everything worked. I ran a few tests and rebooted several times to work out any kinks, and I was good to go.

All pretty routine stuff. At least, until I upgraded to Windows Server 2019...

### The problem after upgrading

As I mentioned above, things started to go haywire. The NextCloud service itself seemed to be OK, and was just acting as the messenger. So immediately after a reboot, I would issue a command to check the iSCSI volume status (results below are a reproduction on a test machine):

```bash
zrasco@zlubuntu-test:/mnt$ sudo iscsiadm -m session -P3
iSCSI Transport Class version 2.0-870
version 2.0-874
Target: iqn.1991-05.com.microsoft:win-q1m7inu0a9c-test-target (non-flash)
        Current Portal: 10.100.101.210:3260,1
        Persistent Portal: 10.100.101.210:3260,1
                **********
                Interface:
                **********
                Iface Name: default
                Iface Transport: tcp
                Iface Initiatorname: iqn.1993-08.org.debian:01:63f27af04bf9
                Iface IPaddress: 10.100.101.212
                Iface HWaddress: <empty>
                Iface Netdev: <empty>
                SID: 1
                iSCSI Connection State: LOGGED IN
                iSCSI Session State: LOGGED_IN
                Internal iscsid Session State: NO CHANGE
                *********
                Timeouts:
                *********
                Recovery Timeout: 120
                Target Reset Timeout: 30
                LUN Reset Timeout: 30
                Abort Timeout: 15
                *****
                CHAP:
                *****
                username: <empty>
                password: ********
                username_in: <empty>
                password_in: ********
                ************************
                Negotiated iSCSI params:
                ************************
                HeaderDigest: None
                DataDigest: None
                MaxRecvDataSegmentLength: 262144
                MaxXmitDataSegmentLength: 65536
                FirstBurstLength: 65536
                MaxBurstLength: 262144
                ImmediateData: Yes
                InitialR2T: No
                MaxOutstandingR2T: 1
                ************************
                Attached SCSI devices:
                ************************
                Host Number: 33 State: running
                scsi33 Channel 00 Id 0 Lun: 0
                        Attached scsi disk sdb          State: running

zrasco@zlubuntu-test:/mnt$
```

Everything seems OK for awhile. But after a few hours of NextCloud activity and getting the errors, I ran the same command again:

```bash
zrasco@zlubuntu-test:/mnt/disk$ sudo iscsiadm -m session -P3
iSCSI Transport Class version 2.0-870
version 2.0-874

...
<Same stuff>
...
                Attached SCSI devices:
                ************************
                Host Number: 33 State: running
                scsi33 Channel 00 Id 0 Lun: 0
                        Attached scsi disk sdb          State: offline

```

But strangely, the drive was still mounted. The dmesg command reported a bunch of stuff similar to this:

```bash
[  760.795417] sd 33:0:0:0: Device offlined - not ready after error recovery
[  760.795421] sd 33:0:0:0: Device offlined - not ready after error recovery
[  760.795446] sd 33:0:0:0: [sdb] tag#0 FAILED Result: hostbyte=DID_ERROR driverbyte=DRIVER_OK
[  760.795449] sd 33:0:0:0: [sdb] tag#0 CDB: Write(10) 2a 00 00 13 c2 00 00 08 40 00
[  760.795455] print_req_error: I/O error, dev sdb, sector 1294848
[  760.795515] EXT4-fs warning (device sdb1): ext4_end_bio:323: I/O error 10 writing to inode 12 (offset 503316480 size 8388608 starting block 162048)
[  760.795521] Buffer I/O error on device sdb1, logical block 161280
[  760.795578] Buffer I/O error on device sdb1, logical block 161281
[  760.795604] Buffer I/O error on device sdb1, logical block 161282
[  760.795629] Buffer I/O error on device sdb1, logical block 161283
[  760.795654] Buffer I/O error on device sdb1, logical block 161284
[  760.795679] Buffer I/O error on device sdb1, logical block 161285
[  760.795704] Buffer I/O error on device sdb1, logical block 161286
[  760.795729] Buffer I/O error on device sdb1, logical block 161287
[  760.795754] Buffer I/O error on device sdb1, logical block 161288
[  760.795779] Buffer I/O error on device sdb1, logical block 161289
[  760.795939] EXT4-fs warning (device sdb1): ext4_end_bio:323: I/O error 10 writing to inode 12 (offset 511705088 size 294912 starting block 162120)
[  760.795981] sd 33:0:0:0: rejecting I/O to offline device
```

It seemed that even though the iSCSI target was offline, the drive was still mounted in some intermediate state. Attempts to work with the directory and device were a disaster:

```bash
zrasco@zlubuntu-test:/mnt/disk$ ls
ls: reading directory '.': Input/output error
zrasco@zlubuntu-test:/mnt/disk$ sudo fdisk /dev/sdb

Welcome to fdisk (util-linux 2.31.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

fdisk: cannot open /dev/sdb: No such device or address

zrasco@zlubuntu-test:/mnt/disk$
```

The only way to get the iSCSI target back was to log off and log back onto the iSCSI target, then remount. So I decided to run a throughput test, and it failed:

```bash
root@zlubuntu-test:/mnt/disk# dd if=/dev/zero of=./test1.img bs=128MB count=30 oflag=dsync status=progress
384000000 bytes (384 MB, 366 MiB) copied, 4 s, 101 MB/s
dd: error writing './test1.img': Input/output error
4+0 records in
3+0 records out
384000000 bytes (384 MB, 366 MiB) copied, 242.003 s, 1.6 MB/s
root@zlubuntu-test:/mnt/disk# ls
ls: reading directory '.': Input/output error
root@zlubuntu-test:/mnt/disk#
```

At least now I had a reliable way to reproduce the issue.

### The fix

Remembering that everything worked fine on my 2012 R2 server, I decided to spin up a test instance of Windows Server 2012 R2 and point my iSCSI initiator to it. This experiment proved to be successful: the throughput tests passed with flying colors. It seemed to be an issue with Windows Server 2019, and possibly Windows Server 2016 as well (which I didn't test).

After some research, I found a [Microsoft technet post](https://social.technet.microsoft.com/Forums/en-US/df227b41-7d56-4fef-b389-d8e190c1ce13/microsoft-iscsi-software-target-on-windows-server-2016-and-openiscsi-initiator-compatibility-issues?forum=winserverfiles) which pointed me in the right direction. Long story short, it turned out to be a bug in open-iscsi which was caused by the underlying block size being incorrectly set to 1280. After some investigating, I discovered this bug was fixed in the scsi driver for Linux 4.17 and above. As you may recall, I was using Linux 4.15.

At first, I was just going to update the kernel to 4.17. But then I discovered that Ubuntu has what's called a Hardware Enablement stack, which allows you to keep your current version of Ubuntu, but upgrade the kernel and hardware information to a later release. You can find the instructions for [your LTS release here](https://wiki.ubuntu.com/Kernel/LTSEnablementStack).

So I ran the command, rebooted, confirmed the new kernel was in-place, and ran my throughput tests again:

```bash
zrasco@zlubuntu-test:/mnt$ sudo apt-get install --install-recommends linux-generic-hwe-18.04
...
zrasco@zlubuntu-test:/mnt$ sudo reboot
...
zrasco@zlubuntu-test:/mnt$ uname -rs
Linux 5.4.0-47-generic
zrasco@zlubuntu-test:/mnt$ dd if=/dev/zero of=./test1.img bs=128MB count=30 oflag=dsync status=progress
3840000000 bytes (3.8 GB, 3.6 GiB) copied, 37 s, 104 MB/s
30+0 records in
30+0 records out
3840000000 bytes (3.8 GB, 3.6 GiB) copied, 37.0255 s, 104 MB/s
zrasco@zlubuntu-test:/mnt$
```

It worked! I ran a few other tests to make sure my other stuff was up and running, and it all looks good so far. Obviously, upgrading the kernel up a major release and all the associated hardware info along with it isn't without risk, but in my case, i did resolve my issue.

Thanks for reading and hope you found this helpful.