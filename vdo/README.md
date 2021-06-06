# VDO workshop

## Requirements

A host where you can ssh without password to the RHEL8's root user.

A RHEL 8 with the following:

| **Requirement** | Lab |
|-----------------|-----|
| __/tmp/__ with at least 15G free | **VDO** |
| /dev/vdb (80GB) | **VDO** |
| /dev/vdc (80GB) | **VDO** |

## Deploying the lab

To deploy the lab:

```bash
[root@hostname ansible]# ansible-playbook -i hosts prepare-rhel8-labs.yml --tags vdo
```

## Reseting the lab

To reset the lab:

```bash
[root@hostname ansible]# ansible-playbook -i hosts reset-rhel8-labs.yml --tags vdo
```

This playbook will:

* Unmount the filesystems for the **VDO** and the logical volume.
* Both volume groups will be removed.
* The **VDO** volume will be removed.