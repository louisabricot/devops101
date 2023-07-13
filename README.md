# Prerequisites

To run this project you will need Vagrant installed on your host machine.

# Work environment setup with Ansible and Vagrant

To set up a development environment with Ansible for a Kubernetes cluster, we use Vagrant to create and provision multiple virtual machines. The architecture consists of five VMs: one for the Ansible Controller Node and four for the Kubernetes cluster (one master and three workers)

## Overview of the architecture

![Architecture Overview](./assets/architecture-overview.png)

## Create Linux Virtual Machine automatically with Vagrant

Vagrant is a tool for automating the creation and management of virtual machines. It relies on providers like VirtualBox to leverage their virtualisation capabilities and simplifies the setup process.

We create the VMs using Vagrant and VirtualBox as the provider. The Vagrantfile contains configuration details for each node.  

```ruby
```

For the controller node, it sets up the VM's resources, hostname, IP address, and port forwarding. Ansible is installed on the controller node using a shell provisioner. For the worker nodes, similar configurations are set up, but Ansible is not installed.

#### Setting up SSH communication between our VMs

To enable a remote configuration our worker nodes from our Ansible controller, we establish SSH communication from the controller to the worker nodes by sharing its public key with the workers.

In the controller's script, we add the following lines to:
- create a *.ssh* directory
- change its owner and group to 'vagrant' (as we are running the script as *root*)
- generate a 4096-bit RSA key (as per ANSSI recommendations :nerd_face:),
- copy the key on our host machine and finally,
- we disable Ansible host key checking to keep this setup automated

```bash
node.vm.provision "shell", inline: <<-SHELL
  # These lines follow the installation of Ansible
  mkdir -p /home/vagrant/.ssh
  chown -R vagrant:vagrant /home/vagrant/.ssh
  chmod 700 /home/vagrant/.ssh
  chmod 600 /home/vagrant/.ssh/authorized_keys
  yes | ssh-keygen -t rsa -b 2048 -f /home/vagrant/.ssh/id_rsa -q -N ""
  cat /home/vagrant/.ssh/id_rsa.pub > /vagrant/controller#{i}_pubkey
  echo "export ANSIBLE_HOST_KEY_CHECKING=False" >> /home/vagrant/.bashrc
  systemctl restart sshd
SHELL
```

Finally, on workers' script, we add the controller's public key to their *authorized_keys*:

```bash
node.vm.provision "shell", inline: <<-SHELL
  # These lines follow the software update
  mkdir -p /home/vagrant/.ssh
  cat /vagrant/controller1_pubkey >> /home/vagrant/.ssh/authorized_keys
  chown -R vagrant:vagrant /home/vagrant/.ssh
  chmod 700 /home/vagrant/.ssh
  chmod 600 /home/vagrant/.ssh/authorized_keys
  systemctl restart sshd
SHELL
```

> **Warning:** This might not be the most secure way but I struggled to find a better strategy and decided to move on to the next step of the project.

### Meta: Folder structure
tree .
vagrant doc
https://developer.hashicorp.com/vagrant/docs/provisioning/ansible_intro

## Configuring our Kubernetes cluster with Ansible

## Inventory file
## Kube master

## Kube workers
# Resources

- [Vagrant for beginners, a tutorial](https://dev.to/kennibravo/vagrant-for-beginners-getting-started-with-examples-jlm)
- [Setting a Kubernetes cluster with Ansible](https://vrukshalitorawane.medium.com/kubernetes-setup-with-wordpress-using-ansible-48dea03dc339)
