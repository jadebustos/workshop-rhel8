# Performance Co-Pilot (WIP)

## Requirements

A host to collect data and at least one server to send data.

## Deploying the lab

Add to the inventory:

```
[[pcp-server]
192.168.100.50

[pcp-clients]
192.168.100.51
192.168.100.52
```

In the variables file for the ansible playbook fill it with your data:

```
pcp_server: ['pcopilot.labmad.redhat.com']
pcp_clients: ['pcopilot-client1.labmad.redhat.com', 'pcopilot-client2.labmad.redhat.com']
```

To deploy the server:

```bash
[root@hostname ansible]# ansible-playbook -i hosts -l pcp-server prepare-rhel8-labs.yml --tags pcp-server
```

To deploy the clients:

```bash
[root@hostname ansible]# ansible-playbook -i hosts -l pcp-clients prepare-rhel8-labs.yml --tags pcp-client
```

## Deleting the lab

Not yet implemented.
```