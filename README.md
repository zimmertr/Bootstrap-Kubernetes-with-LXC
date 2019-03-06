## Bootstrap Kubernetes with LXC

WARNING: This project is no longer developed as I have moved to using QEMU instead. https://github.com/zimmertr/Bootstrap-Kubernetes-with-QEMU

# Summary
Build a 4 node Kubernetes cluster on a Proxmox cluster using Ansible and LXC. 

Approximate deployment time: 20 minutes.

WARNING: See problems section before using this repository.

# Requirements
1. Proxmox server
2. DNS Server
3. Ansible 2.7.0+. Known incompatibility with a previous build. 

# Instructions
1. Modify the `vars.yml` file with values specific to your environment.
2. Provision DNS A records for the IP Addresses & Hostnames you defined for your nodes in the `vars.yml` file.
3. Modify the `inventory.ini` file to reflect your chosen DNS records and the location of the SSH keys used to connect to the nodes.
4. Run the deployment: `ansible-playbook -e @vars.yml -i inventory.ini site.yml`
5. After deployment, a `~/.kube` directory will be created on your workstation. Within your `config` and an `authentication_token` file can be be found. This token is used to authenticate against the Kubernetes API and Dashboard using your account. To connect to the dashboard, install `kubectl` on your workstation and run `kubectl proxy` then navigate to the [Dashboard Endpoint](http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/) in your browser.


# Tips
1. You can rollback the entire deployment with: `ansible-playbook -e @vars.yml -i inventory.ini delete_all_resources.yml`
2. If your LXC instances fail to install `openssh-server` and throw a long `yum` related error, it's likely that they do not have a properly configured network. You can troubleshoot this by using the `lxc-attach` command to connect to them from Promxox without SSH. 
3. See [this repository](https://github.com/zimmertr/Bootstrap-Kubernetes-with-QEMU) to do this with QEMU instead.  Benefits of using QEMU include:
```
* More security since the compute resources aren't sharing kernel space with your server.
* Not at the mercy of the Proxmox kernel for compatibility with necessary Kubernetes kernel modules.
```

# TODO
1. Add better support for multi-node Proxmox clusters.
2. Add support for VLAN Tags & IDs.
3. Perform security audit and enhance if necessary.
4. Rewrite `deploy_lxc_containers.yml` to deploy one instance and clone rather than four separate instances to reduce duration.

# Problems

1) There is a bug in either the `4.15.18` Linux kernel or in the `br_netfilter` module. Preventing the LXC strategy from being a viable solution due to pod networking never being able to work. More information can be found here: https://github.com/lxc/lxd/issues/5193#issuecomment-431872713A A cluster can still be provisioned without pod networking, for what it is worth. 
2. The `k8s` module does not support applying Kubernetes Deployments from URL. Instead of using `get_url` to download them first, and then apply them with `k8s`, I just use `shell` to run a `kubectl apply -f`. [Feature Request here](https://github.com/ansible/ansible/issues/48402).
