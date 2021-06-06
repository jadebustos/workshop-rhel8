# Comparative VDO vs NON VDO

To see the advantages of using VDO we are going to create a VDO volume mounted on ``/media/vdo`` and a standard LVM logical volume mounted on ``/media/lvm``.

We will download an untar several linux kernel source. As there are a lot of shared code and files are text a high-ratio of deduplication and compression will be shown.

We can download kernels from [https://mirrors.edge.kernel.org/pub/linux/kernel/v4.x/](https://mirrors.edge.kernel.org/pub/linux/kernel/v4.x/).

The downloaded kernels:

```
kernels: ['linux-4.0.7.tar.gz', 'linux-4.10.13.tar.gz', 'linux-4.10.3.tar.gz', 'linux-4.1.16.tar.gz', 'linux-4.1.22.tar.gz', 'linux-4.1.29.tar.gz', 'linux-4.1.35.tar.gz', 'linux-4.1.41.tar.gz', 'linux-4.1.48.tar.gz', 'linux-4.1.6.tar.gz', 'linux-4.0.1.tar.gz', 'linux-4.0.8.tar.gz', 'linux-4.10.14.tar.gz', 'linux-4.1.10.tar.gz', 'linux-4.1.17.tar.gz', 'linux-4.1.23.tar.gz', 'linux-4.1.2.tar.gz', 'linux-4.1.36.tar.gz', 'linux-4.1.42.tar.gz', 'linux-4.1.49.tar.gz', 'linux-4.1.7.tar.gz', 'linux-4.0.2.tar.gz', 'linux-4.0.9.tar.gz', 'linux-4.10.15.tar.gz', 'linux-4.1.11.tar.gz', 'linux-4.1.18.tar.gz', 'linux-4.1.24.tar.gz', 'linux-4.1.30.tar.gz', 'linux-4.1.37.tar.gz', 'linux-4.1.43.tar.gz', 'linux-4.1.4.tar.gz', 'linux-4.1.8.tar.gz', 'linux-4.0.3.tar.gz', 'linux-4.0.tar.gz', 'linux-4.10.16.tar.gz', 'linux-4.1.12.tar.gz', 'linux-4.1.19.tar.gz', 'linux-4.1.25.tar.gz', 'linux-4.1.31.tar.gz', 'linux-4.1.38.tar.gz', 'linux-4.1.44.tar.gz', 'linux-4.1.50.tar.gz', 'linux-4.1.9.tar.gz', 'linux-4.0.4.tar.gz', 'linux-4.10.10.tar.gz', 'linux-4.10.17.tar.gz', 'linux-4.1.13.tar.gz', 'linux-4.1.1.tar.gz', 'linux-4.1.26.tar.gz', 'linux-4.1.32.tar.gz', 'linux-4.1.39.tar.gz', 'linux-4.1.45.tar.gz', 'linux-4.1.51.tar.gz', 'linux-4.1.tar.gz', 'linux-4.0.5.tar.gz', 'linux-4.10.11.tar.gz', 'linux-4.10.1.tar.gz', 'linux-4.1.14.tar.gz', 'linux-4.1.20.tar.gz', 'linux-4.1.27.tar.gz', 'linux-4.1.33.tar.gz', 'linux-4.1.3.tar.gz', 'linux-4.1.46.tar.gz', 'linux-4.1.52.tar.gz', 'linux-4.0.6.tar.gz', 'linux-4.10.12.tar.gz', 'linux-4.10.2.tar.gz', 'linux-4.1.15.tar.gz', 'linux-4.1.21.tar.gz', 'linux-4.1.28.tar.gz', 'linux-4.1.34.tar.gz', 'linux-4.1.40.tar.gz', 'linux-4.1.47.tar.gz', 'linux-4.1.5.tar.gz']
```

We will create a 65 GB **VDO** volume on ``/dev/vdb`` mapped as ``/dev/mapper/vdob`` and mounted on ``/media/vdo``.

We will create a 65 GB **logical volume** on ``/dev/vdc`` and mounted on ``/media/lvm``.

We will download serveral linux kernel sources in ``/tmp/kernels`` and we will untar them on ``/media/vdo`` and ``/media/lvm``.

```bash
[root@rhel8 ~]# df -hP
Filesystem                   Size  Used Avail Use% Mounted on
devtmpfs                     1.9G     0  1.9G   0% /dev
tmpfs                        1.9G     0  1.9G   0% /dev/shm
tmpfs                        1.9G  8.7M  1.9G   1% /run
tmpfs                        1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/mapper/rhel-root         47G  8.8G   39G  19% /
/dev/vda1                   1014M  315M  700M  31% /boot
/dev/mapper/temp-temp         11G  9.0G  2.1G  82% /tmp/kernels
tmpfs                        379M     0  379M   0% /run/user/0
/dev/mapper/vdoVG-vdoLV       65G   50G   16G  77% /media/vdo
/dev/mapper/novdoVG-novdoLV   65G   50G   16G  77% /media/lvm
[root@rhel8 ~]# ls /media/vdo | wc -l
73
[root@rhel8 ~]# ls /media/lvm | wc -l
73
[root@rhel8 ~]#
```

Apparently the space used in both mount points is the same.

Checking **VDO**:

```bash
[root@rhel8 ~]# vdostats --human-readable
Device                    Size      Used Available Use% Space saving%
/dev/mapper/vdob         70.0G      5.8G     64.2G   8%           96%
[root@rhel8 ~]#
```

So with **VDO**:

* Only 5.8G are being used instead of 50G.
* 5.8G used from the 70G available on ``/dev/vdb`` (8%).
* **VDO** compression and deduplication is saving 96% space.

