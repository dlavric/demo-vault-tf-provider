# -*- mode: ruby -*-
# vi: set ft=ruby :

# download Vault by specifying the version
VAULT=""

Vagrant.configure("2") do |config|
    config.vm.box = "hashicorp/bionic64"

    

    config.vm.define "vault" do |vault|
        vault.vm.hostname = "vault"
        vault.vm.network "private_network", ip: "192.168.56.41"
        vault.vm.provider "virtualbox" do |v|     
            v.memory = 1024 * 4
            v.cpus = 2  
    end

    vault.vm.provision "shell", path: "scripts/install_vault.sh",
      env: { "VAULT" => VAULT||=String.new }
    
    end  

# Install Terraform

    config.vm.provision "shell", path: "scripts/install_terraform.sh"
   
    
end