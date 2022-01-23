# Playbooks for managing my K3S Pi cluster

## Initial setup

Raspbian default configuration is nice but not perfect:
```
ansible-playbook raspberrypi-initial-setup.yaml -K
```

## Shutdown cluster

```
ansible-playbook shutdown-nodes.yaml -K
```

## Install K3S

Yeah, this easy:

```
ansible-playbook install-k3s.yaml -K
```
