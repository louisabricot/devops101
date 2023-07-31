NUM_NODES = 2
NUM_CONTROLLER_NODE = 1
IP_NTW = "192.168.56."
CONTROLLER_IP_START = 2
NODE_IP_START = CONTROLLER_IP_START + NUM_CONTROLLER_NODE

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"

  # Controller Nodes
  (1..NUM_CONTROLLER_NODE).each do |i|
    config.vm.define "controller#{i}" do |node|
      node.vm.provider "virtualbox" do |vb|
        vb.name = "controller#{i}"
        vb.memory = 2048
        vb.cpus = 2
      end

      node.vm.hostname = "controller#{i}"
      node.vm.network "private_network", ip: "#{IP_NTW}#{CONTROLLER_IP_START + i}"
      node.vm.network "forwarded_port", guest: 22, host: "#{2710 + i}"
      node.vm.provision "shell", inline: <<-SHELL

        # Software update & install Ansible
        add-apt-repository --yes --update ppa:ansible/ansible
        apt-get update
        apt-get install software-properties-common -y
        apt-get install ansible -y

        # Generate SSH public key and copy it on host machine
        mkdir -p /home/vagrant/.ssh
        yes | ssh-keygen -t rsa -b 2048 -f /home/vagrant/.ssh/id_rsa -q -N ""
        chown -R vagrant:vagrant /home/vagrant/.ssh
        chmod 700 /home/vagrant/.ssh
        chmod 600 /home/vagrant/.ssh/authorized_keys
        cat /home/vagrant/.ssh/id_rsa.pub > /vagrant/controller#{i}_pubkey
        systemctl restart sshd

        # Generate inventory file
        echo "[kubemasters]" > /home/vagrant/workstation/inventory.ini
        for i in {1..1}
        do
            echo "kubemaster$i ansible_host=192.168.56.$(( 3 + $i ))" >> /home/vagrant/workstation/inventory.ini
        done
        echo "[kubeworkers]" >> /home/vagrant/workstation/inventory.ini
        for i in {2..#{NUM_NODES}}
        do
            echo "kubeworker$(( $i - 1)) ansible_host=192.168.56.$(( 3 + $i ))" >> /home/vagrant/workstation/inventory.ini
            done
      SHELL

       # Synchronize inventory.ini file
      node.vm.synced_folder "./provisioning", "/home/vagrant/workstation"
    end
  end

  # Worker Nodes
  (1..NUM_NODES).each do |i|
    config.vm.define "worker#{i}" do |node|
      node.vm.provider "virtualbox" do |vb|
        vb.name = "worker#{i}"
        vb.memory = 2048
        vb.cpus = 2
      end

      node.vm.hostname = "worker#{i}"
      node.vm.network "private_network", ip: "#{IP_NTW}#{NODE_IP_START + i}"
      node.vm.network "forwarded_port", guest: 22, host: "#{2720 + i}"
      node.vm.provision "shell", inline: <<-SHELL
        apt-get update

        # Adds controller public key to worker's authorized_keys
        mkdir -p /home/vagrant/.ssh
        cat /vagrant/controller1_pubkey >> /home/vagrant/.ssh/authorized_keys
        chown -R vagrant:vagrant /home/vagrant/.ssh
        chmod 700 /home/vagrant/.ssh
        chmod 600 /home/vagrant/.ssh/authorized_keys
        systemctl restart sshd

      SHELL
    end
  end
end
