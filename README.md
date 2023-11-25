# K3S Pi Cluster

The [monitoring stack](https://github.com/carlosedp/cluster-monitoring) used is the [Carlos Eduardo](https://github.com/carlosedp) version of the [kube-prometheus](https://github.com/prometheus-operator/kube-prometheus) repo and part of the Ansible roles were adapted from [Jeff Geerling](https://github.com/geerlingguy) [turing-pi-cluster](https://github.com/geerlingguy/turing-pi-cluster) project.

## What does the playbook do?

This Playbook do a couple of things:
- Perform initial setup in the cluster nodes
- Install K3S in a cluster with 1 master and n workers nodes (Adapt `hosts.ini` to your needs)
- Install an NFS server to allow the cluster to provide persistent volumes to pods through a NFS provider.
- Install an NFS provider and Prometheus - AlertManager - Grafana as monitoring stack out of the box
- Install cert-manager with some letsencrypt issuers to allow the creation of valid https certificates

## Compatibility

This configuration was tested with the following Raspberry Pi and OS combinations:
- Raspberry Pi 4 model B and Raspberry Pi OS (32-bit)
- Raspberry Pi 4 model B and Raspberry Pi OS (64-bit)

Other configurations may work but you know, they have not been tested.

## Requisites

- You need Ansible of course
- The master node should have Golang installed, since manual installation is needed to get an updated version I let this to you :)
- The Ansible collections in the requirements file: `ansible-galaxy install -r requirements.yaml`
- All your Raspberrys should have a valid network configuration with static IPs and hostname and SSH access configured with private key.

## Configuration

You can tweak the next variables under the `group_vars` folder:
- `timezone`: Timezone that will be configured in the cluster nodes
- `nfs_share`: NFS share path (Avoid locations that need `root` access)
- `suffix_domain`: The domain suffix to use in the Grafana, Prometheus and AlertManager ingresses
- `cluster_monitoring_version`: The version of the cluster-monitoring repo to check out
- `cluster_monitoring_update_repo`: If you change the `cluster_monitoring_version` above set this to true to force the update
- `grafana_from_email`: The admin email used in Grafana
- `certmanager_version`: cert-manager version to install
- `cloudflare_email` and `cloudflare_token`: If you set this two variables the playbook will install a cluster issuer that will use Cloudflare API for letsencrypt certificates instead of the http challenge
- `letsencrypt_email`: The email to use for the letsencrypt certificates
- `externalTrafficPolicy`: Let you decide what policy the Traefik load balancer should follow. [More information](https://kubernetes.io/docs/tasks/access-application-cluster/create-external-load-balancer/#preserving-the-client-source-ip)

## Usage

Just execute the playbook and go for a coffee:
```
ansible-playbook main.yaml -K
```
After the installation you can find the cluster `kubeconfig` file in the master node under: `/etc/rancher/k3s/k3s.yaml`.

If you only need to execute part of it you can use the next tags (The names are self explanatory):
- `pi-initial-setup`
- `install-k3s-master`
- `install-k3s-workers`
- `install-nfs-server`
- `basic-cluster-setup`
- `install-cert-manager`

Also, it is recomended to change the next line in `/var/lib/rancher/k3s/server/manifests/local-storage.yaml` after installation:

```yaml
storageclass.kubernetes.io/is-default-class: "true" # Set this to false to make sure you use NFS as the default storage class
```

## Exposing your cluster to the Internet

**NOTE:** `externalTrafficPolicy` must be configured to `Local` in order for this steps to work

To expose our ingresses to the internet we need to prepare some things to avoid problems. Since we are going to have private services that can be reached through an ingress we need to create a traefik middleware for all the ingresses we want to make private to prevent traffic from the internet to go to them:
```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: private
  namespace: kube-system
spec:
  ipWhiteList:
    sourceRange:
      - 127.0.0.1/32
      - 10.0.0.0/8
      - 172.16.0.0/12
      - 192.168.0.0/16
```

Once we have that in the cluster the only thing left is asking traefik to use it where we want. Just add this to the metadata section of the Ingresses you don’t want to be accesible from the internet: 
```yaml
annotations:
    traefik.ingress.kubernetes.io/router.middlewares: kube-system-private@kubernetescrd
```

## Extra

There is an extra playbook called `shutdown-nodes.yaml` that will just connect to every node in the cluster to shut it down. Useful to power off the cluster completely in a safer way:
```
ansible-playbook shutdown-nodes.yaml -K
```

## Troubleshooting

- If the Grafana pod is not starting as it should, check that the `rpc-statd.service` service is running in the NFS server host.
