# Work environment setup with Ansible and Vagrant

## Overview
localhost -> vagrant up
ansible control node -> ansible-playbook ... 
kubernetes cluster :
- kubemaster
- kubeworkers

## Create Linux Virtual Machine automatically with Vagrant

goals: 
- one "Ansible Control Node" machine from which we run the ansible scripts to configure our "workers" machine
- see ansible documentation to set the control node
- multiple machine creation with Vagrantfile

- why automate this ? As we are learning to configure and provision machines we might break things

### Setting up the Ansible Control Node

vagrant file


### Setting up SSH communication between our VMs

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

