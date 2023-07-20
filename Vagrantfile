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
    chown -R vagrant:vagrant /home/vagrant
    export DEBIAN_FRONTEND=noninteractive
    apt-get update && apt-get upgrade -y && apt-get dist-upgrade -y    
    apt-get install -y ansible
  SHELL

  config.vm.provision "ansible_local" do |ansible|
    ansible.become = true
    ansible.playbook = "ansible/site.yml"
    ansible.inventory_path = "ansible/production.ini"
    ansible.limit = "localhost"
  end
end
