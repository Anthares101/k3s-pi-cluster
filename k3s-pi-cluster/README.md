# Playbooks for managing my K3S Pi cluster

## Initial setup

Raspbian default configuration is nice but not perfect:
```
ansible-playbook raspberrypi-initial-setup.yaml -i hosts -K
```

## Shutdown cluster

```
ansible-playbook shutdown-nodes.yaml -i hosts -K
```
