# Ansible playbooks to deploy labs

This playbooks are used to deploy labs.

## Requirements

A host where you can ssh without password to the RHEL8's root user.

## Deploying the labs

Two playbooks to deploy the labs:

* [prepare-rhel8-labs.yml](prepare-rhel8-labs.yml) which deploy the labs.
* [reset-rhel8-labs.yml](reset-rhel8-labs.yml) which reset the lab as if was freshly deployed.

There are some common tasks to all labs as disabling **firewalld** or create users that are carried out by a specific role, so you will have to run:

```bash
[root@hostname ansible]# ansible-playbook -i hosts prepare-rhel8-labs.yml --tags common
```

> ![ANSIBLE](../imgs/tip-icon.png) **TIP**: Do it before deploying any lab.

In each lab you could see how to deploy only that lab. However, if you want to deploy all labs:

```bash
[root@hostname ansible]# ansible-playbook -i hosts prepare-rhel8-labs.yml --tags full
```

> ![ANSIBLE](../imgs/tip-icon.png) **TIP**: If .

You can use several tags with both playbooks:

* ``full`` which will deploy/reset all labs.
* A specific tag for each lab that you can check in each lab's section.