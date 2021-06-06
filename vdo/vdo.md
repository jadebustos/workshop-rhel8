# VDO

Virtual Data Optimizer (VDO) provides inline data reduction for Linux in the form of deduplication, compression, and thin provisioning.

## Installing VDO

```bash
[root@rhel8 ~]# yum install vdo kmod-kvdo
```

## Creating a VDO volume

As an example we will create a LVM logical volume on top of a VDO volume.

First a VDO volume has to be created:

```bash
[root@rhel8 ~]# vdo create --name=vdob --device=vdb --vdoLogicalSize=15G
```

Check the status:

```bash
[root@rhel8 ~]# vdo status --name=vdob
VDO status:
  Date: '2019-10-14 15:45:20+02:00'
  Node: rhel8.backend.lab
Kernel module:
  Loaded: true
  Name: kvdo
  Version information:
    kvdo version: 6.2.0.293
Configuration:
  File: /etc/vdoconf.yml
  Last modified: '2019-10-14 15:43:47'
VDOs:
  vdob:
    Acknowledgement threads: 1
    Activate: enabled
    Bio rotation interval: 64
    Bio submission threads: 4
    Block map cache size: 128M
    Block map period: 16380
    Block size: 4096
    CPU-work threads: 2
    Compression: enabled
    Configured write policy: auto
    Deduplication: enabled
    Device mapper status: 0 41943040 vdo /dev/vdb normal - online online 1051492 5242880
    Emulate 512 byte: disabled
    Hash zone threads: 1
    Index checkpoint frequency: 0
    Index memory setting: 0.25
    Index parallel factor: 0
    Index sparse: disabled
    Index status: online
    Logical size: 20G
    Logical threads: 1
    Max discard size: 4K
    Physical size: 20G
    Physical threads: 1
    Slab size: 2G
    Storage device: /dev/vdb
    VDO statistics:
      /dev/mapper/vdob:
        1K-blocks: 20971520
        1K-blocks available: 16765552
        1K-blocks used: 4205968
        512 byte emulation: false
...
        write amplification ratio: 0.0
        write policy: async
[root@rhel8 ~]# 
```

After that we will create the logical volume on top of the VDO volume:

```bash
[root@rhel8 ~]# pvcreate /dev/mapper/vdbo
  Physical volume "/dev/mapper/vdbo" successfully created.
[root@rhel8 ~]# vgcreate vdoVG /dev/mapper/vdbo
  Volume group "vdbVG" successfully created
[root@rhel8 ~]# lvcreate -L15G -n vdoLV /dev/mapper/vdbo
  Logical volume "vdoLV" created.
[root@rhel8 ~]#
```

Now we have to create the filesystem:

```bash
[root@rhel8 ~]# mkfs.xfs -K -i size=512 /dev/vdoVG/vdoLV
meta-data=/dev/temp/temp         isize=512    agcount=4, agsize=983040 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1
data     =                       bsize=4096   blocks=3932160, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
[root@rhel8 ~]# xfs_info /dev/mapper/vdoVG-vdoLV 
meta-data=/dev/mapper/vdoVG-vdoLV isize=512    agcount=4, agsize=983040 blks
         =                       sectsz=4096  attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1
data     =                       bsize=4096   blocks=3932160, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=4096  sunit=1 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
[root@rhel8 ~]#
```

To mount on boot the filesystem you will have to add to ``/etc/fstab``:

```
/dev/vdoVG/vdoLV /media/vdo xfs x-systemd.requires=vdo.service 0 0
```