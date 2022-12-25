# simple-fcos-cluster
A simple kubernetes cluster using fcos, kvm and k0sctl

## Dependencies

- [proxmox-ve](https://www.proxmox.com/en/proxmox-ve)
- [terraform](https://www.terraform.io/)
- [xz](https://en.wikipedia.org/wiki/XZ_Utils)
- [k0sctl](https://github.com/k0sproject/k0sctl)
- [haproxy](http://www.haproxy.org/)

## One-time Configuration

### Create a tfvars file

```
cp terraform.tfvars.example terraform.tfvars
# Edit and save the variables according to your liking
vim terraform.tfvars
```

### Create an HA Proxy Server

This is a manual step. I set this up on my Raspberry Pi. You can choose to do the same in a LXC container or a VM.

Make sure that the path to the config is always `/etc/haproxy/haproxy.cfg` and make sure that the service is enabled.

```
<!-- This step will change based on your package manager -->
apt-get install haproxy
systemctl enable haproxy
systemctl start haproxy
```


### Creating the cluster

```
terraform init -upgrade
terraform plan
# WARNING: The next command will override ~/.kube/config. Make a backup if needed.
terraform apply --auto-approve
```

### What does 'terraform apply' do?

- Downloads a version of fcos depending on the tfvars
- Converts the zipped file to qcow2 and moves it to proxmox node
- Creates a template using the qcow2 image
- Creates ignition files for each VM
- Creates nodes with the ignition configurations and other params as specified in the tfvars
- Updates the haproxy configuration on a VM/raspberry pi
- Creates a k0s cluster when the nodes are ready
- Replaces ~/.kube/config with the new kubeconfig from k0sctl


#### TODO

- Automate DHCP IP reservation


#### Debugging HA Proxy

```
haproxy -c -f /etc/haproxy/haproxy.cfg
```

#### What about libvirt?

There is a branch named ['kvm'](https://github.com/Naman1997/simple-fcos-cluster/tree/kvm) in the repo that has steps to create a similar cluster using the 'dmacvicar/libvirt' provider. I won't be maintaining that branch - but it can be used as a frame of reference for someone who wants to create a fcos based k8s cluser in their homelab.