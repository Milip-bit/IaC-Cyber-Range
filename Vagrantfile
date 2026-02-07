# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  
  config.vm.define "soc-server" do |soc|
    # --- 1. System ---
    soc.vm.box = "bento/ubuntu-22.04"
    soc.vm.hostname = "soc-server"

    # --- 2. Network (Port Forwarding) ---
    soc.vm.network "forwarded_port", guest: 8000, host: 8000
    soc.vm.network "forwarded_port", guest: 9997, host: 9997
    
    # Static IP for communication with machines within the lab
    soc.vm.network "private_network", ip: "10.0.1.10"

    # --- 3. Resources ---
    soc.vm.provider "vmware_desktop" do |v|
      v.gui = false
      v.memory = 4096
      v.cpus = 2
      v.allowlist_verified = true
    end

    # --- 4. Installation of docker and setup of Splunk container (Bash) ---
    soc.vm.provision "shell", inline: <<-SHELL
      export DEBIAN_FRONTEND=noninteractive

      if ! command -v docker &> /dev/null; then
          apt-get update
          apt-get install -y docker.io
          usermod -aG docker vagrant
      fi

      docker rm -f splunk || true

      docker run -d \
        -p 8000:8000 \
        -p 9997:9997 \
        -e "SPLUNK_GENERAL_TERMS=--accept-sgt-current-at-splunk-com" \
        -e "SPLUNK_START_ARGS=--accept-license" \
        -e "SPLUNK_PASSWORD=CyberLab123!" \
        --name splunk \
        --restart unless-stopped \
        splunk/splunk:latest
    SHELL
  end
end