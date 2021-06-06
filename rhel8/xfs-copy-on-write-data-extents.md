# XFS copy-on-write data extents

This feature allows files to share data common blocks.

To see if we can take advantage of this new feature the ``reflink`` attribute has to be set to 1. This is the default in **RHEL 8**: 

```bash
[root@rhel8 ~]# xfs_info /
meta-data=/dev/mapper/rhel-root  isize=512    agcount=4, agsize=1113856 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1
data     =                       bsize=4096   blocks=4455424, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
[root@rhel8 ~]# 
```

We will create a 2 GB file:

```bash
[root@rhel8 ~]# df -hP
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               1.4G     0  1.4G   0% /dev
tmpfs                  1.4G     0  1.4G   0% /dev/shm
tmpfs                  1.4G  8.6M  1.4G   1% /run
tmpfs                  1.4G     0  1.4G   0% /sys/fs/cgroup
/dev/mapper/rhel-root   17G  1.8G   16G  11% /
/dev/vda1             1014M  333M  682M  33% /boot
tmpfs                  278M     0  278M   0% /run/user/0
[root@rhel8 ~]# dd if=/dev/urandom of=data.bin bs=1M count=2048
2048+0 records in
2048+0 records out
2147483648 bytes (2.1 GB, 2.0 GiB) copied, 36.7265 s, 58.5 MB/s
[root@rhel8 ~]# ls -lh
total 2.1G
-rw-------. 1 root root 1.5K May 30 12:12 anaconda-ks.cfg
-rw-r--r--. 1 root root 2.0G Oct 10 15:37 data.bin
drwxr-xr-x. 2 root root   74 May 30 12:11 openscap_data
[root@rhel8 ~]# df -hP
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               1.4G     0  1.4G   0% /dev
tmpfs                  1.4G     0  1.4G   0% /dev/shm
tmpfs                  1.4G  8.6M  1.4G   1% /run
tmpfs                  1.4G     0  1.4G   0% /sys/fs/cgroup
/dev/mapper/rhel-root   17G  3.8G   14G  22% /
/dev/vda1             1014M  333M  682M  33% /boot
tmpfs                  278M     0  278M   0% /run/user/
[root@rhel8 ~]# 
```

Now we copy the file to a new file as we usually do:

```bash
[root@rhel8~]# cp data.bin data.bin_cp 
[root@rhel8 ~]# ls -l
total 4194308
-rw-------. 1 root root       1471 May 30 12:12 anaconda-ks.cfg
-rw-r--r--. 1 root root 2147483648 Oct 10 15:37 data.bin
-rw-r--r--. 1 root root 2147483648 Oct 10 15:38 data.bin_cp
drwxr-xr-x. 2 root root         74 May 30 12:11 openscap_data
[root@rhel8 ~]# df -hP
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               1.4G     0  1.4G   0% /dev
tmpfs                  1.4G     0  1.4G   0% /dev/shm
tmpfs                  1.4G  8.6M  1.4G   1% /run
tmpfs                  1.4G     0  1.4G   0% /sys/fs/cgroup
/dev/mapper/rhel-root   17G  5.8G   12G  34% /
/dev/vda1             1014M  333M  682M  33% /boot
tmpfs                  278M     0  278M   0% /run/user/0
[root@rhel8 ~]#
```

As we can see we have now two copies of the same file and the storage has decreased in 2G after we copied the file, as expected.

> ![NOTE](../imgs/note-icon.png) All the blocks were duplicated, that's why it took some time to finish the cp command.

No we copy the file to a new file but using the ``--reflink`` option:

```bash
[root@rhel8 ~]# df -hP
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               1.4G     0  1.4G   0% /dev
tmpfs                  1.4G     0  1.4G   0% /dev/shm
tmpfs                  1.4G  8.6M  1.4G   1% /run
tmpfs                  1.4G     0  1.4G   0% /sys/fs/cgroup
/dev/mapper/rhel-root   17G  5.8G   12G  34% /
/dev/vda1             1014M  333M  682M  33% /boot
tmpfs                  278M     0  278M   0% /run/user/0
[root@rhel8 ~]# cp --reflink data.bin data.bin_cp_reflink
[root@rhel8 ~]# ls -l
total 6291460
-rw-------. 1 root root       1471 may 30 12:12 anaconda-ks.cfg
-rw-r--r--. 1 root root 2147483648 oct 10 15:37 data.bin
-rw-r--r--. 1 root root 2147483648 oct 10 15:38 data.bin_cp
-rw-r--r--. 1 root root 2147483648 oct 10 16:06 data.bin_cp_reflink
drwxr-xr-x. 2 root root         74 may 30 12:11 openscap_data
[root@rhel8 ~]# df -hP
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               1.4G     0  1.4G   0% /dev
tmpfs                  1.4G     0  1.4G   0% /dev/shm
tmpfs                  1.4G  8.6M  1.4G   1% /run
tmpfs                  1.4G     0  1.4G   0% /sys/fs/cgroup
/dev/mapper/rhel-root   17G  5.8G   12G  34% /
/dev/vda1             1014M  333M  682M  33% /boot
tmpfs                  278M     0  278M   0% /run/user/0
[root@rhel8 ~]# md5sum data.bin*
b47e1e4a694a9e3292c86d9f090e3713  data.bin
b47e1e4a694a9e3292c86d9f090e3713  data.bin_cp
b47e1e4a694a9e3292c86d9f090e3713  data.bin_cp_reflink
[root@rhel8 ~]# 
```

We can see that:

* There are three copies of the same file.
* There was no additional space consumed after the third copy was done.
* The ``cp`` command did not take time to finish.

We are going to check block mapings for the original file and the file copied without ``--reflink``:

```bash
[root@rhel8 ~]# xfs_bmap -v data.bin data.bin_cp
data.bin:
 EXT: FILE-OFFSET         BLOCK-RANGE        AG AG-OFFSET            TOTAL
   0: [0..2097023]:       18470088..20567111  2 (648392..2745415)  2097024 100000
   1: [2097024..4194303]: 20567456..22664735  2 (2745760..4843039) 2097280 100000
data.bin_cp:
 EXT: FILE-OFFSET         BLOCK-RANGE        AG AG-OFFSET            TOTAL
   0: [0..2097023]:       22664736..24761759  2 (4843040..6940063) 2097024
   1: [2097024..4194303]: 27622416..29719695  3 (889872..2987151)  2097280
[root@rhel8 ~]# 
```

As we can see the block mappins are different for the two files, that mean that there are two copies of the file in different blocks.

Now with the one using ``--reflink``:

```bash
[root@rhel8 ~]# xfs_bmap -v data.bin data.bin_cp_reflink 
data.bin:
 EXT: FILE-OFFSET         BLOCK-RANGE        AG AG-OFFSET            TOTAL
   0: [0..2097023]:       18470088..20567111  2 (648392..2745415)  2097024 100000
   1: [2097024..4194303]: 20567456..22664735  2 (2745760..4843039) 2097280 100000
data.bin_cp_reflink:
 EXT: FILE-OFFSET         BLOCK-RANGE        AG AG-OFFSET            TOTAL
   0: [0..2097023]:       18470088..20567111  2 (648392..2745415)  2097024 100000
   1: [2097024..4194303]: 20567456..22664735  2 (2745760..4843039) 2097280 100000
[root@rhel8 ~]#
```

As we can see both files use the same blocks so there is only one copy of the file.

Now we are going to modify the file copied with ``--reflink`` inserting data into the file:

```bash
[root@rhel8 ~]# printf 'selling vauxhall corsa' | LANG=C dd of=data.bin_cp_reflink bs=1M seek=1024 count=22 conv=notrunc
0+1 records in
0+1 records out
22 bytes copied, 0.00126724 s, 17.4 kB/s
[root@rhel8 ~]# ls -l
total 6291588
-rw-------. 1 root root       1471 may 30 12:12 anaconda-ks.cfg
-rw-r--r--. 1 root root 2147483648 oct 10 15:37 data.bin
-rw-r--r--. 1 root root 2147483648 oct 10 15:38 data.bin_cp
-rw-r--r--. 1 root root 2147483648 oct 10 16:06 data.bin_cp_reflink
drwxr-xr-x. 2 root root         74 may 30 12:11 openscap_data
[root@rhel8 ~]# df -hP
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               1.4G     0  1.4G   0% /dev
tmpfs                  1.4G     0  1.4G   0% /dev/shm
tmpfs                  1.4G  8.6M  1.4G   1% /run
tmpfs                  1.4G     0  1.4G   0% /sys/fs/cgroup
/dev/mapper/rhel-root   17G  5.8G   12G  34% /
/dev/vda1             1014M  333M  682M  33% /boot
tmpfs                  278M     0  278M   0% /run/user/0
[root@rhel8 ~]# md5sum data.bin*
b47e1e4a694a9e3292c86d9f090e3713  data.bin
b47e1e4a694a9e3292c86d9f090e3713  data.bin_cp
1b586e41d96d2c44db0db97e641a121d  data.bin_cp_reflink
[root@rhel8 ~]# 
```

As we can see the ``--reflink`` has been modified.

Now we are going to check the block mappings:

```bash
[root@rhel8 ~]# xfs_bmap -v data.bin data.bin_cp_reflink 
data.bin:
 EXT: FILE-OFFSET         BLOCK-RANGE        AG AG-OFFSET            TOTAL
   0: [0..2097023]:       18470088..20567111  2 (648392..2745415)  2097024 100000
   1: [2097024..2097151]: 20567456..20567583  2 (2745760..2745887)     128 100000
   2: [2097152..2097159]: 20567584..20567591  2 (2745888..2745895)       8
   3: [2097160..4194303]: 20567592..22664735  2 (2745896..4843039) 2097144 100000
data.bin_cp_reflink:
 EXT: FILE-OFFSET         BLOCK-RANGE        AG AG-OFFSET            TOTAL
   0: [0..2097023]:       18470088..20567111  2 (648392..2745415)  2097024 100000
   1: [2097024..2097151]: 20567456..20567583  2 (2745760..2745887)     128 100000
   2: [2097152..2097159]: 17870360..17870367  2 (48664..48671)           8
   3: [2097160..4194303]: 20567592..22664735  2 (2745896..4843039) 2097144 100000
[root@rhel8 ~]# 
```

Now the block mappings are different. We can obserse that there are some blockmappings which both files have in common but there are others that are not the same. There is only one copy of the common information:

```
0: [0..2097023]:       18470088..20567111  2 (648392..2745415)  2097024 100000
1: [2097024..2097151]: 20567456..20567583  2 (2745760..2745887)     128 100000
3: [2097160..4194303]: 20567592..22664735  2 (2745896..4843039) 2097144 100000
```

And there are two copies of the blocks which are not the same for both files. Only blocks for the changes will be allocated.