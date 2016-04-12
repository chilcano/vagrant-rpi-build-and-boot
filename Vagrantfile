# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  # https://atlas.hashicorp.com/ubuntu/boxes/trusty64 [Official Ubuntu Server 14.04 LTS (Trusty Tahr) builds]
  config.vm.box = "ubuntu/trusty64"
  config.vm.provider "virtualbox" do |vb|
    config.vbguest.auto_update = true
    vb.customize ["modifyvm", :id, "--memory", "6144"]
    vb.customize ["modifyvm", :id, "--cpus", "4"]    
  end
  # If you want to use this system to netboot Raspberry Pi, then uncomment this line
  #config.vm.network "public_network", bridge: "en4: mac-eth0", ip: "10.0.0.1"
  config.vm.network "public_network", bridge: "ask", ip: "10.0.0.1"
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbook.yml"
  end
end
