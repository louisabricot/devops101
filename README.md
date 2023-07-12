# Prerequisites
- vagrant
- provider

# Work environment setup with Ansible and Vagrant

For this project, we will need to create 5 virtual machines: 
- one VM will be used to run our Ansible configuration (Ansible Controller Node)
- four VMs will make up the k8s cluster (1 master node and 3 workers nodes)

![Architecture Overview](./assets/architecture-overview.png)

## Create Linux Virtual Machine automatically with Vagrant

Vagrant is a tool for automating the creation and management of virtual machines. It relies on providers like VirtualBox to leverage their virtualisation capabilities and simplifies the setup process.

We will use a Vagrantfile to define our multiple Linux machines, set the SSH communication between them and install all the required software packages to our controller. 

### Vagrantfile

### Setting up SSH communication between our VMs

To enable SSH communication between the Ansible controller and the virtual machines dedicated to the Kubernetes cluster
goals:
- we want to configure our worker01 and worker02 from our nodecontroller with ansible
- to do this, ansible establishes an SSH communication so let's make sure it is properly configure
generating 2048 RSA keys for SSH as per ANSSI guidance
Warning: this gave me quite a lot of trouble and I am not sure it was the best approach

### Inventory file 

Dynamic creation of inventory file ?

### Meta: Folder structure
tree .
vagrant doc
https://developer.hashicorp.com/vagrant/docs/provisioning/ansible_intro

## Configuring our Kubernetes cluster with Ansible

## Kube master

## Kube workers
# Resources

- [Vagrant for beginners, a tutorial](https://dev.to/kennibravo/vagrant-for-beginners-getting-started-with-examples-jlm)
- [Setting a Kubernetes cluster with Ansible](https://vrukshalitorawane.medium.com/kubernetes-setup-with-wordpress-using-ansible-48dea03dc339)
