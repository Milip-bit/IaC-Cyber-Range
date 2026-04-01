# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  
  # --- SOC SPLUNK SERVER ---
  config.vm.define "soc-server" do |soc|
    soc.vm.box = "Home-lab-boxes/soc-server"
    soc.vm.hostname = "soc-server"
    soc.vm.boot_timeout = 600
    soc.vm.network "forwarded_port", guest: 8000, host: 8000
    soc.vm.network "forwarded_port", guest: 9997, host: 9997
    soc.vm.network "private_network", ip: "10.0.1.10"

    soc.vm.provider "virtualbox" do |vb|
      vb.gui = false
      vb.memory = "4096"
      vb.cpus = 2
    end

  end

  # --- 2.WINDOWS 10 VICTIM MACHINE ---
  config.vm.define "victim-pc" do |win|
    win.vm.box = "Home-lab-boxes/victim-pc"
    win.vm.hostname = "victim-pc"
    win.vm.communicator = "winrm"
    win.vm.boot_timeout = 1200 

    win.vm.network "private_network", ip: "10.0.1.20"

    win.vm.provider "virtualbox" do |vb|
      vb.gui = true
      vb.memory = "12288"
      vb.cpus = 6
      vb.customize ["modifyvm", :id, "--accelerate3d", "on"]
      vb.customize ["modifyvm", :id, "--graphicscontroller", "vboxsvga"]
      vb.customize ["modifyvm", :id, "--vram", "256"]
      vb.customize ["modifyvm", :id, "--paravirtprovider", "default"]
      vb.customize ["modifyvm", :id, "--clipboard-mode", "bidirectional"]
      vb.customize ["modifyvm", :id, "--draganddrop", "bidirectional"]
    end

  end
end