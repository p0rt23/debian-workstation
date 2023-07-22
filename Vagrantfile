# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "debian/bookworm64"
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
    vb.cpus = "1"
  end

  config.vm.hostname = "debian-bookworm"
  config.vm.synced_folder ".", "/vagrant", type: "virtualbox"
  config.vm.synced_folder "../../Projects", "/home/vagrant/projects", type: "virtualbox"
  
  config.vm.provision "file", source: "~/.ssh", destination: "~/.ssh"

  config.vm.provision "shell", inline: <<-SHELL
    export DEBIAN_FRONTEND=noninteractive
    apt-get update && apt-get upgrade -y && apt-get dist-upgrade -y    
    apt-get install -y ansible
    ansible-galaxy install -r /home/vagrant/projects/debian-workstation/ansible/roles/requirements.yml
  SHELL

  config.vm.provision "ansible_local" do |ansible|
    ansible.become = true
    ansible.playbook = "ansible/site.yml"
    ansible.inventory_path = "ansible/production.ini"
    ansible.limit = "localhost"
  end

  config.vm.provision "shell", inline: <<-SHELL
    find /home/vagrant -path /home/vagrant/projects -prune -false -o -name '*' -exec chown vagrant:vagrant {} ';'
  SHELL
end
