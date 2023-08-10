# Automated Kubernetes Cluster Setup using Vagrant and Ansible

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Setting up the Development Environment](#setting-up-the-development-environment)
3. [Automatically Creating Virtual Machines with Vagrant](#automatically-creating-virtual-machines-with-vagrant)
4. [Configuring a Kubernetes Cluster with Ansible](#configuring-a-kubernetes-cluster-with-ansible)
    - [Ansible Configuration File (ansible.cfg)](#ansible-configuration-file)
    - [Ansible Inventory File (inventory.ini)](#ansible-inventory-file)
    - [Utilizing Ansible Roles](#utilizing-ansible-roles)
5. [Deploying Wordpress and MySQL with persistent volumes](#deploying-wordpress-and-mysql-with-persistent-volumes)

## Prerequisites

Before proceeding with the setup, ensure you have the following prerequisites on your host machine:
- **Vagrant** installed, and
- **VirtualBox** (or any other Vagrant provider) installed for virtualization capabilities.

## Setting up the development environment

We used Vagrant to create our development environment for this project. Vagrant helps us automate the creation and provisioning of multiple virtual machines from one configuration file: the *Vagrantfile*.

### Architecture Overview

The overall architecture consists of our host machine and five VMs: one for the Ansible Controller Node and four for the Kubernetes cluster:
- Ansible Controller Node (1 VM)
- Kubernetes Cluster (1 Master Node and 3 Worker Nodes)

![Overview of the architecture](./assets/architecture-overview.png)

## Automatically Creating Virtual Machines with Vagrant

Vagrant is a tool for automating the creation and management of virtual machines. It relies on providers like VirtualBox to leverage their virtualization capabilities and simplifies the setup process.

In the Vagrantfile, we define two types of nodes:
- *controller* nodes refers to the Ansible Control Node,
- *worker* nodes are part of the Kubernetes cluster.

Both types of nodes share similar OS distribution and general resources such as CPUs and memory. Additionally, they are configured to be in the same private network to facilitate communication. For the controller node, Ansible is installed using a shell provisioner.

### Setting up SSH communication between our VMs

To set up SSH communication between the VMs, we ensure the controller's SSH public key is shared with the worker nodes. This allows the controller to remotely configure the worker nodes.

The following snippets illustrate the SSH setup process in the Vagrantfile:

#### Controller node

```ruby
# Vagrantfile
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

#### Worker nodes

Finally, in the workers' script, we add the controller's public key to their *authorized_keys*:

```ruby
# Vagrantfile
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

To enhance our working environment, we share the project's repository with our controller
node:

```ruby
node.vm.synced_folder "./provisioning", "/home/vagrant/workstation"
```

By synchronizing the *provisioning* directory with "/home/vagrant/workstation" on the controller node, we can easily access and manage our Ansible playbooks and configuration files.

## Configuring a Kubernetes cluster with Ansible

Ansible is an open-source automation tool that efficiently configures, manages, and deploys applications and infrastructure. To customize Ansible's behavior, we use configuration files.

### Ansible configuration file

```cfg
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

### Ansible inventory file

The inventory file specifies the hosts and groups that Ansible will manage during playbook execution. We dynamically generate this file in the [Vagrantfile](./Vagrantfile) to ensure it always contains the IP addresses of the virtual machines, irrespective of the number of nodes provisioned.

```ini
# inventory.ini
[kubemasters]
kubemaster1 ansible_host=192.168.56.4
[kubeworkers]
kubeworker1 ansible_host=192.168.56.5
kubeworker2 ansible_host=192.168.56.6
kubeworker3 ansible_host=192.168.56.7
```

> :warning: The script to generate the inventory file is very basic: it only sets one kubemaster and does not check the value of the `NUM_NODES` variable.

### Utilizing Ansible Roles

Ansible roles organize and package automation tasks, variables, and configurations into a reusable unit. By breaking down the Kubernetes cluster setup into roles, it becomes easier to manage and update specific components or configurations independently, promoting consistency and simplifying deployment across different environments.

To start, we define three roles, corresponding to our inventory groups:
- *k8s_master* (corresponding to the kubemaster role) installs Docker, configures Kubernetes core components, Calico network, and generates a token for workers to join the cluster.
- *k8s_worker* (corresponding to the kuberworker role) installs Docker and joins the Kubernetes cluster.
- and finally, the *common* role stores tasks that are common to both *k8s_master* and *k8s_worker* roles.

```markdown
.
├── common
├── k8s_master
└── k8s_worker
```
#### Common

The *common* role's primary purpose is to encapsulate tasks that are common to both *k8s_master* and *k8s_worker*. 
They share two steps: setting up Docker and Kubernetes.

Rather than replicating the same set of tasks in each individual role, *k8s_master* and *k8s_worker* can simply invoke the *common* role. This not only makes the automation process more modular but also enhances maintainability and adaptability.

##### Setting up Docker

Docker is a pre-requisite for installing Kubernetes
talk about containers

##### Setting up Kubernetes

talk about kubernetes

talk about the CRI error with containerd 

#### K8s_master role

The *tasks/main.yml* file orchestrates the different subtasks in the following order:

```markdown
---
- name: Setup Docker
  include_role:
    name: common
    tasks_from: docker_setup

- name: Setup Kubernetes
  include_role:
    name: common
    tasks_from: kubernetes_setup

- name: Initialize Kubernetes cluster
  include_tasks: init_kubernetes_cluster.yml

- name: Calico network setup
  include_tasks: calico_setup.yml

- name: Generate join command and copy to local file
  include_tasks: join_command.yml
```
##### Initializing the Kubernetes cluster

`KUBELET_EXTRA_ARGS=`
`kubeadm init`

##### Setting up the Calico network

- what is Calico network

##### Creating the join command

- token
  
#### K8s_worker role

Once the Kubernetes cluster is started, we can create our k8s_workers. To do so, the *tasks/main.yml* file orchestrates the different subtasks in the following order:

```markdown
---
- name: Setup Docker
  include_role:
    name: common
    tasks_from: docker_setup

- name: Setup Kubernetes
  include_role:
    name: common
    tasks_from: kubernetes_setup

- name: Configure node ip
  include_tasks: configure_node.yml

- name: Join command
  include_tasks: join_command.yml
```

As we previously explained, the first two tasks are common to the k8_master role.

##### Configuring our worker node

`kubelet KUBELET_EXTRA_ARGS`

##### Joining the Kubernetes cluster

`join_command.sh`

To set up our Kubernetes cluster, we connect to our *AnsibleControlNode1* machine and run:

```bash
cd workstation
ansible-playbook setup.yml
```
`ansible-playbook` implicitly takes the inventory file by resolving its location from the inventory_path variable set in the *ansible.cfg* configuration file. The *setup.yml* file shows the order in which our three roles must be applied.

Once the configuration has ended, we can check that our Kubernetes cluster was successfully created by connecting to our master node and getting some information about our cluster:

```bash
ssh 192.168.56.4
kubectl get nodes -o wide -n kube-system
kubectl get nodes -o wide
kubectl get pods
kubectl get services
```

We are now ready to add another role to run Wordpress and MySQL in our cluster.

### Deploying Wordpress and MySQL with persistent volumes

To deploy a Wordpress 

#### Persistent volumes and claims

#### Secrets and configmap

#### MySQL service and deployment

#### Wordpress service and deployment
