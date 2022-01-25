# K3S Pi Cluster

The [monitoring stack](https://github.com/carlosedp/cluster-monitoring) used is the [Carlos Eduardo](https://github.com/carlosedp) version of the [kube-prometheus](https://github.com/prometheus-operator/kube-prometheus) repo and part of the Ansible roles were adapted from [Jeff Geerling](https://github.com/geerlingguy) [turing-pi-cluster](https://github.com/geerlingguy/turing-pi-cluster) project.

## What does the playbook do?

This Playbook do a couple of things:
- Perform initial setup in the cluster nodes
- Install K3S in a cluster with 1 master and n workers nodes (Adapt `hosts.ini` to your needs)
- Install an NFS server to allow the cluster to provide Persistent volumes to pods through a NFS provider.
- Install an NFS provider in the cluster
- Install Prometheus - AlertManager - Grafana as monitoring stack out of the box

## Compatibility

This configuration was tested with the following Raspberry Pi and OS combinations:
- Raspberry Pi 4 model B and Raspberry Pi OS (32-bit)
- Raspberry Pi 4 model B and Raspberry Pi OS (64-bit)

Other configurations may work but you know, the have not been tested.

## Requisites

- You need Ansible of course
- The Ansible collections in the requirements file: `ansible-galaxy install -r requirements.yaml`
- All you Raspberrys should have a valid network configuration with static IPs and hostname and SSH access configured with private key.

## Configuration

You can tweak the next variables under the `group_vars` folder:
- `timezone`: Timezone that will be configured in the cluster nodes
- `nfs_share`: NFS share path (Avoid locations that need `root` access)
- `suffix_domain`: The domain suffix to use in the Grafana, Prometheus and AlertManager ingresses
- `cluster_monitoring_version`: The version of the cluster-monitoring repo to check out
- `cluster_monitoring_update_repo`: If you change the `cluster_monitoring_version` above set this to true to force the update
- `grafana_from_email`: The admin email used in Grafana

## Usage

Just execute the playbook and go for a coffee:
```
ansible-playbook main.yaml -K
```
If you only need to execute part of it you can use the next tags (The names are self explanatory):
- `pi-initial-setup`
- `install-k3s-master`
- `install-k3s-workers`
- `install-nfs-server`
- `configure-cluster`

## Extra

There is a extra playbook called `shutdown-nodes.yaml` that will just connect to every node in the cluster to shut it down. Useful to power off the cluster completely in a safer way:
```
ansible-playbook shutdown-nodes.yaml -K
```
