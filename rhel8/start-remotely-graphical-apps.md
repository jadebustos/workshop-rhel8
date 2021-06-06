# Start graphical apps remotely

To be able to start graphical applications in one RHEL 8 server and using a remote X server to display you need to install the following packages in the RHEL 8 server:

```bash
[root@rhel8 ~]# yum install xorg-x11-xauth xorg-x11-server-utils bitmap-fixed-fonts xorg-x11-fonts-75dpi.noarch xorg-x11-fonts-100dpi.noarch xorg-x11-fonts-misc.noarch
```

You need to set the following values in ``/etc/ssh/sshd_config``:

```
X11Forwarding yes
X11UseLocalhost yes
```

Restart the **sshd** daemon if necessary.

Then you have to connect to the RHEL 8 server from the host you have a X Server running:

```bash
[root@Xserver ~]# ssh -X root@rhel8
Last login: Thu Oct 10 21:11:34 2019 from 192.168.200.1
/usr/bin/xauth:  file /root/.Xauthority does not exist
[root@rhel8 ~]# xhost +
access control disabled, clients can connect from any host
xhost:  must be on local machine to enable or disable access control.
[root@rhel8 ~]#
```

And you can start now your graphical application which will use the remote X server.