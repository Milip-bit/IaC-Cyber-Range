# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  
  config.vm.define "soc-server" do |soc|
    soc.vm.box = "bento/ubuntu-22.04"
    soc.vm.hostname = "soc-server"
    soc.vm.network "forwarded_port", guest: 8000, host: 8000
    soc.vm.network "forwarded_port", guest: 9997, host: 9997
    soc.vm.network "private_network", ip: "10.0.1.10"

    soc.vm.provider "virtualbox" do |vb|
      vb.gui = false
      vb.memory = "4096"
      vb.cpus = 2
    end

    soc.vm.provision "shell", inline: <<-SHELL
      export DEBIAN_FRONTEND=noninteractive
      if ! command -v docker &> /dev/null; then
          apt-get update && apt-get install -y docker.io
          usermod -aG docker vagrant
      fi
      docker rm -f splunk || true
      docker run -d -p 8000:8000 -p 9997:9997 \
        -e "SPLUNK_GENERAL_TERMS=--accept-sgt-current-at-splunk-com" \
        -e "SPLUNK_START_ARGS=--accept-license" \
        -e "SPLUNK_PASSWORD=CyberLab123!" \
        --name splunk --restart unless-stopped \
        splunk/splunk:latest
    SHELL
  end

  config.vm.define "victim-pc" do |win|
    win.vm.box = "gusztavvargadr/windows-10"
    win.vm.hostname = "victim-pc"
    win.vm.communicator = "winrm"
    
    win.vm.boot_timeout = 1200 

    win.vm.network "private_network", ip: "10.0.1.20"

    win.vm.provider "virtualbox" do |vb|
      vb.gui = true
      vb.memory = "8192"
      vb.cpus = 4
      vb.customize ["modifyvm", :id, "--accelerate3d", "on"]
      vb.customize ["modifyvm", :id, "--graphicscontroller", "vboxsvga"]
      vb.customize ["modifyvm", :id, "--vram", "256"]
      vb.customize ["modifyvm", :id, "--paravirtprovider", "default"]
    end

    win.vm.provision "shell", inline: <<-SHELL
      Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
      Test-Connection -ComputerName 10.0.1.10 -Count 4
    SHELL
  end
end