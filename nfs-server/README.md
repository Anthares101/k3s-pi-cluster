# Install NFS server

Just edit the `vars.yml` to configure the installation and NFS share:

- nfs_share: Path to the directory to share
- exports_line: The line that will be added to the NFS `exports` file in order to share a directory 

Once everythong is configured launch the playbook!
```
ansible-playbook -K install-nfs-server.yaml
```
