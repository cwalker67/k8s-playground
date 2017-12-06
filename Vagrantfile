# -*- mode: ruby -*-
# # vi: set ft=ruby :

Vagrant.configure(2) do |config|

  config.vm.define "k8s-master" do |s|
    s.ssh.forward_agent = true
    s.vm.box = "ubuntu/xenial64"
    s.vm.hostname = "k8s-master"
    s.vm.provision :shell, path: "scripts/bootstrap_ansible.sh"
    s.vm.provision :shell, inline: "PYTHONUNBUFFERED=1 ansible-playbook /vagrant/ansible/k8s-master.yml -c local"
    s.vm.network "private_network", ip: "172.42.42.100", netmask: "255.255.255.0",
      auto_config: true,
      virtualbox__intnet: "k8s-net"
    s.vm.provider "virtualbox" do |v|
      v.name = "k8s-master"
      v.memory = 1024
      v.gui = false
    end
  end

  (1..3).each do |i|
    config.vm.define "k8s-node-#{i}" do |s|
      s.ssh.forward_agent = true
      s.vm.box = "ubuntu/xenial64"
      s.vm.hostname = "k8s-node-#{i}"
      s.vm.provision :shell, path: "scripts/bootstrap_ansible.sh"
      s.vm.provision :shell, inline: "PYTHONUNBUFFERED=1 ansible-playbook /vagrant/ansible/k8s-worker.yml -c local"
      s.vm.network "private_network", ip: "172.42.42.10#{i}", netmask: "255.255.255.0",
        auto_config: true,
        virtualbox__intnet: "k8s-net"
      s.vm.provider "virtualbox" do |v|
        v.name = "k8s-node-#{i}"
        v.memory = 1024
        v.gui = false
      end
    end
  end

end
