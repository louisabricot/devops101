# Prerequisites

To run this project, you will need Vagrant installed on your host machine.

# Setting up our development environment

We used Vagrant to create our development environment for this project. Vagrant helps us automate the creation and provisioning of multiple virtual machines from one configuration file: the *Vagrantfile*.

The overall architecture consists of our host machine and five VMs: one for the Ansible Controller Node and four for the Kubernetes cluster (one master and three workers).

Overview of the architecture
-

![Overview of the architecture](./assets/architecture-overview.png)

## Creating Virtual Machines automatically with Vagrant

Vagrant is a tool for automating the creation and management of virtual machines. It relies on providers like VirtualBox to leverage their virtualization capabilities and simplifies the setup process.

In the Vagrantfile, we define two types of nodes:
- *controller* nodes refers to the Ansible Control Node,
- *worker* nodes are part of the Kubernetes cluster.

Both types of nodes share similar OS distribution and general resources such as CPUs and memory. There are also configured to be in the same private network. For the controller node, Ansible is installed using a shell provisioner.

## Setting up SSH communication between our VMs

To enable a remote configuration the worker nodes from the controller, we establish SSH communication by sharing the controller's SSH public key with the workers.

In the controller's script, after the installation of Ansible, we add the following commands:

```ruby
node.vm.provision "shell", inline: <<-SHELL

 # These lines follow the installation of Ansible

 # Creates .ssh directory with proper owner, group and permissions
 mkdir -p /home/vagrant/.ssh
 chown -R vagrant:vagrant /home/vagrant/.ssh
 chmod 700 /home/vagrant/.ssh
 chmod 600 /home/vagrant/.ssh/authorized_keys

 # Generates a 4096-bit RSA key
 yes | ssh-keygen -t rsa -b 2048 -f /home/vagrant/.ssh/id_rsa -q -N ""

 # Copies the key on the host machine
 cat /home/vagrant/.ssh/id_rsa.pub > /vagrant/controller#{i}_pubkey

 # Restarts the SSH daemon just in case
 systemctl restart sshd

SHELL
```

Finally, in the workers' script, we add the controller's public key to their *authorized_keys*:

```ruby
node.vm.provision "shell", inline: <<-SHELL
  # These lines follow the software update

  # Creates a .ssh directory with proper owner, group and permissions
  mkdir -p /home/vagrant/.ssh

  # Adds the controller1's public key to the worker's authorized keys
  cat /vagrant/controller1_pubkey >> /home/vagrant/.ssh/authorized_keys

  chown -R vagrant:vagrant /home/vagrant/.ssh
  chmod 700 /home/vagrant/.ssh
  chmod 600 /home/vagrant/.ssh/authorized_keys

  # Restarts the SSH daemon just in case
  systemctl restart sshd
SHELL
```

To improve our working environment, we share the repository to our controller
node.

```ruby
node.vm.synced_folder "./provisioning", "/home/vagrant/workstation"
```

We are ready !

# Configuring a Kubernetes cluster with Ansible

Ansible is an open-source automation tool that allows us to efficiently configure, manage, and deploy applications and infrastructure.

## Ansible configuration file

Overall, this configuration file customizes Ansible's behavior by specifying the inventory file path, setting default options for SSH connections and privilege escalation, making the automation process more efficient and suitable for our environment.

```bash
# ansible.cfg
[defaults]
inventory=/home/vagrant/inventory.ini            # Specify the location of the inventory file
host_key_checking=False                          # Disable confirmation prompt during remote host connection
ask_pass=False                                   # Disable password authentification
remote_user=vagrant                              # Set default remote user to "vagrant"

[privilege_escalation]
become=True                                      # Enable privilege escalation
become_method=sudo                               # Specify privilege escalation method to "sudo"
become_user=root                                 # Set superuser to "root"
become_ask_pass=False                            # Disable password authentication for root
```

## Ansible inventory file
An Ansible inventory file is a text file that contains a list of hosts and groups that Ansible can target and manage during playbook execution.
This inventory file defines two groups:
- *kubemasters* refer to VMs assuming the role of a Kubernetes master node,
- *kubeworkers* refer to VMs assuming the role of a Kubernetes worker node.

```bash
# inventory.ini
[kubemasters]
kubemaster1 ansible_host=192.168.56.4
[kubeworkers]
kubeworker1 ansible_host=192.168.56.5
```

In our [Vagrantfile](./Vagrantfile), we dynamically generate the Ansible inventory file, ensuring it always contains the IP addresses of the virtual machines created by Vagrant, regardless of whether we provision 3 or 7 nodes.

With this inventory file, you can now target the *kubemasters* and *kubeworkers* groups in your Ansible playbooks, making it easy to manage and configure your Kubernetes cluster components separately.

## Roles

Ansible roles organize and package automation tasks, variables, and configurations into a reusable unit. By breaking down the Kubernetes cluster setup into roles, it becomes easier to manage and update specific components or configurations independently, promoting consistency and simplifying deployment across different environments.

To start, we define two roles, corresponding to our inventory groups:
- *kubemaster* installs Docker, configure Kubernetes core components, Calico network, and generates the join command for worker nodes to join the cluster.
- *kubeworker* 


# Resources

- [Vagrant for beginners, a tutorial](https://dev.to/kennibravo/vagrant-for-beginners-getting-started-with-examples-jlm)
- [Setting a Kubernetes cluster with Ansible](https://vrukshalitorawane.medium.com/kubernetes-setup-with-wordpress-using-ansible-48dea03dc339)
