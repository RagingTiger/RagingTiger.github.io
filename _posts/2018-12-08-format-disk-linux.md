---
layout: post
title: "Reformatting Disk Drive on Linux to ext4: parted command"
date: 2018-12-08
category: [file system, hard drives, reformatting, linux, rpi]
---
### <a name="toc"></a> Table of Contents
* [Introduction](#intro)
* [Walkthrough](#walkthru)
  * [Find Device Name](#device)
  * [Parted Command](#parted)
  * [Remove Partition](#delpart)
* [References](#references)

## <a name="intro"></a> [Introduction](#toc)
This walkthrough for reformatting a drive (already formatted) to `ext4` on
*Linux* is inspired heavily from the sources found in the
[reference section](#references). The only difference is some added clarity
and detail for the reformatting process

## <a name="walkthru"></a> [Walkthrough](#toc)
What follows is a short walkthrough for finding a disk connected to the system,
removing a partition, and making a new partition.

### <a name="device"></a> [Find Device Name](#toc)
First, physically connect the drive to the system, either through USB (for
external drives like on the RPi) or through SATA on the motherboard (i.e. for
PCs). Once connected, the device can be found using the `lsblk` command like so:
```
$ lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda           8:0    0  4.6T  0 disk
└─sda1        8:1    0  4.6T  0 part
mmcblk0     179:0    0 59.6G  0 disk
├─mmcblk0p1 179:1    0 41.8M  0 part /boot
└─mmcblk0p2 179:2    0 59.6G  0 part /
```

The above example output was taken from a Raspberry Pi 2. The device labeled
`sda` is a *Seagate 5TB External Hard Drive*, whereas the `mmclk0` is the *micro
SD card* that is formatted with the *Raspibian OS*. The important thing here, is
we now know which device is our Hard Drive.

If we do a quick `ls /dev`, we can see *ALL* devices connected to the system:
```
$ ls /dev
autofs           loop6               ram5     tty18  tty43      uhid
block            loop7               ram6     tty19  tty44      uinput
bsg              loop-control        ram7     tty2   tty45      urandom
btrfs-control    mapper              ram8     tty20  tty46      vchiq
bus              mem                 ram9     tty21  tty47      vcio
cachefiles       memory_bandwidth    random   tty22  tty48      vc-mem
char             mmcblk0             raw      tty23  tty49      vcs
console          mmcblk0p1           rfkill   tty24  tty5       vcs1
cpu_dma_latency  mmcblk0p2           sda      tty25  tty50      vcs2
cuse             mqueue              sda1     tty26  tty51      vcs3
disk             net                 serial0  tty27  tty52      vcs4
fb0              network_latency     sg0      tty28  tty53      vcs5
fd               network_throughput  shm      tty29  tty54      vcs6
full             null                snd      tty3   tty55      vcs7
fuse             ppp                 stderr   tty30  tty56      vcsa
gpiochip0        ptmx                stdin    tty31  tty57      vcsa1
gpiomem          pts                 stdout   tty32  tty58      vcsa2
hwrng            ram0                tty      tty33  tty59      vcsa3
initctl          ram1                tty0     tty34  tty6       vcsa4
input            ram10               tty1     tty35  tty60      vcsa5
kmsg             ram11               tty10    tty36  tty61      vcsa6
log              ram12               tty11    tty37  tty62      vcsa7
loop0            ram13               tty12    tty38  tty63      vcsm
loop1            ram14               tty13    tty39  tty7       vhci
loop2            ram15               tty14    tty4   tty8       watchdog
loop3            ram2                tty15    tty40  tty9       watchdog0
loop4            ram3                tty16    tty41  ttyAMA0    zero
loop5            ram4                tty17    tty42  ttyprintk
```

Without going into great detail about what all these *"devices"* are, we can see
that in the third column our device `sda` is located. **WARNING**: If you have
more than one drive connected, the label they are given may change. This means
that you cannot count on the drive to always be labeled `sda`. If there are two
drives connected, one will be labeled `sda` and the other `sdb`, **BUT** there
is no guarantee which will get which label ... it can change between reboots
etc ... Always check with `lsblk` or other commands (i.e. `sudo fdisk -l`) which
drives have been labeled which device files.

Now we can move on to using the `parted` command.

### <a name="parted"></a> [Parted Command](#toc)[^fn1]
Now that we are clear we have the right drive, fire up an *interactive parted
session* by launching the following (using the device file name we found for our
drive earlier):
```
$ sudo parted /dev/sda
GNU Parted 3.2
Using /dev/sda
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted)
```
If we type help we can see the potential reformatting options:
```
(parted) help
align-check TYPE N                        check partition N for TYPE(min|opt)
        alignment
  help [COMMAND]                           print general help, or help on
        COMMAND
  mklabel,mktable LABEL-TYPE               create a new disklabel (partition
        table)
  mkpart PART-TYPE [FS-TYPE] START END     make a partition
  name NUMBER NAME                         name partition NUMBER as NAME
  print [devices|free|list,all|NUMBER]     display the partition table,
        available devices, free space, all found partitions, or a particular
        partition
  quit                                     exit program
  rescue START END                         rescue a lost partition near START
        and END
  resizepart NUMBER END                    resize partition NUMBER
  rm NUMBER                                delete partition NUMBER
  select DEVICE                            choose the device to edit
  disk_set FLAG STATE                      change the FLAG on selected device
  disk_toggle [FLAG]                       toggle the state of FLAG on selected
        device
  set NUMBER FLAG STATE                    change the FLAG on partition NUMBER
  toggle [NUMBER [FLAG]]                   toggle the state of FLAG on partition
        NUMBER
  unit UNIT                                set the default unit to UNIT
  version                                  display the version number and
        copyright information of GNU Parted
```

### <a name="delpart"></a> [Remove Partition](#toc)
We can see that `rm NUMBER` will delete a partition, but how do we know what
are the partitions? Simple, we run `print` like so:
```
(parted) print
Model: Seagate Expansion Desk (scsi)
Disk /dev/sda: 5001GB
Sector size (logical/physical): 512B/4096B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name     Flags
 1      1049kB  5001GB  5001GB  ext4         primary
```
Here we see that we have **one** partition. To remove it simply type:
```
(parted) rm 1
```
Simply repeat this with all partitions you want to remove (i.e. `rm 2`, `rm 3`).
Now let us look at adding a *new* partition.

### <a name="addpart"></a> [Adding Partition](#toc)[^fn2]
First to add a new partition, we can exit the interactive session and simply
call `parted` and pass the commands to it. We will do this to set the new
partition standard (this assumes you have a fresh drive, or have already removed
all the partitions before using `rm NUMBER`):
```
$ sudo parted /dev/sda mklabel gpt
```
Now we can create a new partition spanning the whole disk:
```
$ sudo parted -a opt /dev/sda mkpart primary ext4 0% 100%
```
This will create one *primary* partition with the `ext4` format (which parted
does not support formatting). Now we are ready to add an `ext4` file system to
this partition.

### <a name="ext4"></a> [Formatting ext4](#toc)
Finally we only need to run a simple command to complete the reformatting of the
drive, with a new partition in the `ext4` format:
```
$ sudo mkfs.ext4 -L OPTIONAL_DISK_NAME /dev/sda1
```
This will create the `ext4` file system on the partition at `/dev/sda1` we
created earlier. The `-L` flag will set the label of the partition to whatever
you set as `OPTIONAL_DISK_NAME`. This is optional and not necessary.

## <a name="sum"></a> [Summary](#toc)
The drive is now completely reformatted with a fresh partition. You can now
mount this drive:
```
$ sudo mount -t ext4 /mnt/DISK_NAME /dev/sda1
```
At last, your new drive is ready to use!

## <a name="references"></a> [References](#toc)
[^fn1]: [Remove Partition Linux](https://serverfault.com/a/749265)
[^fn2]: [Format Drive Linux](https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux)
