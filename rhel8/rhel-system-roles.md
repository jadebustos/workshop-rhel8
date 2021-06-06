# RHEL System Roles

RHEL System Roles is a collection of ansible roles and modules to remotely manage and configure RHEL systems.

They were introduced in RHEL 7.4 and on RHEL 8 the following are available:

* selinux
* kdump
* network
* timesync

> ![INFORMATION](../imgs/information-icon.png) These roles are available through the ``rhel-system-roles`` package available in the AppStream repository.

These roles are designed to manage system configurations across multiple versions of RHEL, as well as adopting new major releases.

## Requirements

A host where you can ssh without password to the RHEL8's root user.

A RHEL 8 with the following:

| **Requirement** | Lab |
|-----------------|-----|
| A non configured network interface | **System Roles** |

## Deploying the lab

You will have to configure ``groups_var/system_role.yml`` to fit your environment:

```yaml
iface: 'enp7s0'
route_metric4: 100
dhcp4: 'no'
dns: '192.168.200.1'
dns_search: 'frontend.lab'
ipaddr: '192.168.100.90'
prefix: 24
network: '192.168.100.0'
gateway: '192.168.100.254'
metric: 4
route_append_only: 'no'
rule_append_only: 'yes'
```

To deploy the lab:

```bash
[root@hostname ansible]# ansible-playbook -i hosts prepare-rhel8-labs.yml --tags system_roles
```

## Reseting the lab

To reset the lab:

```bash
[root@hostname ansible]# ansible-playbook -i hosts reset-rhel8-labs.yml --tags system_roles
```

This playbook will:

* Deactivated the interface which was configured with the system role.
* Remove all configuration files related to that interface.
* Restart network services.

# Lab

## Install rhel system roles

To install the roles:

```bash
[root@rhel8 ~]# yum install rhel-system-roles
```

> ![IMPORTANT](../imgs/important-icon.png) You need to have installed ansible engine.

To install ansible:

```bash
[root@rhel8 ~]# subscription-manager repos --enable ansible-2-for-rhel-8-x86_64-rpms
Repository 'ansible-2-for-rhel-8-x86_64-rpms' is enabled for this system.
[root@rhel8 ~]# 
```

Every role includes a README file with information about how to use it. Documentation can be found in ``/usr/share/doc/rhel-system-roles/SUBSYSTEM/``.

For instance for the network subsystem this file ``/usr/share/doc/rhel-system-roles/network/README.md`` includes all the information about how to use the role.

This is an example about how this can be used to configure the ``enp7s0`` network interface in one host:

```yaml
---

- hosts: rhel8
  vars:
    network_connections:
      - name: enp7s0
        type: ethernet
        ip:
          route_metric4: 100
          dhcp4: no
          dns:
            - 192.168.200.1
          dns_search:
            - frontend.lab
        
          address:
            - 192.168.100.90/24

          route:
            - network: 192.168.150.0
              prefix: 24
              gateway: 198.68.100.254
              metric: 4
          route_append_only: no
          rule_append_only: yes

  roles:
    - role: rhel-system-roles.network 
```

Before running the playbook:

```bash
[root@rhel8 ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:a2:ab:fc brd ff:ff:ff:ff:ff:ff
    inet 192.168.200.90/24 brd 192.168.200.255 scope global noprefixroute enp1s0
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fea2:abfc/64 scope link 
       valid_lft forever preferred_lft forever
3: enp7s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:ce:a1:2a brd ff:ff:ff:ff:ff:ff
[root@rhel8 ~]# 
```
After running the playbook:

```bash
[root@rhel8 ~]# cd demos/system_roles/
[root@rhel8 system_roles]# ansible-playbook -i hosts configure-network.yml 
```

After finishing:

```bash
[root@rhel8 ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:a2:ab:fc brd ff:ff:ff:ff:ff:ff
    inet 192.168.200.90/24 brd 192.168.200.255 scope global noprefixroute enp1s0
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fea2:abfc/64 scope link 
       valid_lft forever preferred_lft forever
3: enp7s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:ce:a1:2a brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.90/24 brd 192.168.100.255 scope global noprefixroute enp7s0
       valid_lft forever preferred_lft forever
    inet6 fe80::4272:e692:239c:193b/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
[root@rhel8 ~]# ip ro
default via 192.168.200.1 dev enp1s0 proto static metric 100 
192.168.100.0/24 dev enp7s0 proto kernel scope link src 192.168.100.90 metric 100 
192.168.150.0/24 via 198.68.100.254 dev enp7s0 proto static metric 4 
192.168.200.0/24 dev enp1s0 proto kernel scope link src 192.168.200.90 metric 100 
198.68.100.254 dev enp7s0 proto static scope link metric 4 
[root@rhel8 ~]# cat /etc/sysconfig/network-scripts/ifcfg-enp7s0 
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=none
IPADDR=192.168.100.90
PREFIX=24
DNS1=192.168.200.1
DOMAIN=frontend.lab
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV4_ROUTE_METRIC=100
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=enp7s0
UUID=b6d70f79-aad1-49ac-b0b2-ffdfc142b541
DEVICE=enp7s0
ONBOOT=yes
[root@rhel8 ~]#
```

> ![INFORMATION](../imgs/information-icon.png) More information in this [Red Hat article](https://access.redhat.com/articles/3050101).