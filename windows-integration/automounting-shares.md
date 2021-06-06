 Mounting windows shares

## Context

IdM has not been deployed due to IBM has not created the required DNS records due to IBM needs to perform a consultancy about that.

So AD clients will join the AD domain using SSSD.

The following procedure has been tested on:

+ RHEL 8.2 joined to an IdM realm which have a trust relationship with an AD.
+ RHEL 8.3 joined to an AD with SSSD.

## Procedure (fstab)

For an user to mount windows shares when login on GDM and umount them when logout add the following to **/etc/fstab**:

```bash
//bbvaad.windows.labmad.redhat.com/ShareDocs /media/share  cifs  rw,noauto,sec=krb5i,nounix,vers=3.1.1,username=user1@windows.labmad.redhat.com,cruid=user1@windows.labmad.redhat.com,domainauto,user,uid=user1@windows.labmad.redhat.com 0 0
```

**cifs-utils** and **keyutils** packages need to be installed.

Grant SUID perms to **mount.cifs**:

```console
[root@cifs ~]# chmod u+s /usr/sbin/mount.cifs
```

To mount the share on user login:

```console
[root@cifs ~]# cat /etc/gdm/PostLogin/Default
#!/bin/sh
#
# Note: this is a sample and will not be run as is.  Change the name of this
# file to <gdmconfdir>/PostLogin/Default for this script to be run.  This
# script will be run before any setup is run on behalf of the user and is
# useful if you for example need to do some setup to create a home directory
# for the user or something like that.  $HOME, $LOGIN and such will all be
# set appropriately and this script is run as root.

mount /media/share
[root@cifs ~]#
```

To umount share on user logout:

``console
[root@cifs ~]# cat /etc/gdm/PostSession/Default
#!/bin/sh

umount /media/share

exit 0
[root@cifs ~]#
```

## Procedure (autofs)

Not done.