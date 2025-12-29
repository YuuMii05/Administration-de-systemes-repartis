Vagrant.configure("2") do |config|
  config.vm.boot_timeout = 1800

  config.vm.provider "vmware_desktop" do |v|
    v.gui = false
    v.allowlist_verified = :disable_warning
  end

  config.vm.define "admin" do |m|
    m.vm.box = "generic/ubuntu2204"
    m.vm.hostname = "admin"
    m.vm.network "private_network", ip: "192.168.77.10"

    m.vm.provider "vmware_desktop" do |v|
      v.vmx["displayname"] = "megaTP-admin"
      v.vmx["memsize"]     = "2048"
      v.vmx["numvcpus"]    = "2"
    end
  end

  config.vm.define "node01" do |m|
    m.vm.box = "generic/rocky9"
    m.vm.hostname = "node01"
    m.vm.network "private_network", ip: "192.168.77.11"

    m.vm.provision "shell", inline: <<-SHELL
      set -eux
      mkdir -p /home/vagrant/.ssh
      echo "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKst079fkB+YnHa35U+Pa+jYyyHRi76hvdJ8p3wmxSA9 vagrant@admin" >> /home/vagrant/.ssh/authorized_keys
      chown -R vagrant:vagrant /home/vagrant/.ssh
      chmod 700 /home/vagrant/.ssh
      chmod 600 /home/vagrant/.ssh/authorized_keys
    SHELL

    m.vm.provider "vmware_desktop" do |v|
      v.vmx["displayname"] = "megaTP-node01"
      v.vmx["memsize"]     = "2048"
      v.vmx["numvcpus"]    = "2"
    end
  end

  config.vm.define "node02" do |m|
    m.vm.box = "generic/rocky9"
    m.vm.hostname = "node02"
    m.vm.network "private_network", ip: "192.168.77.12"

    m.vm.provision "shell", inline: <<-SHELL
      set -eux
      mkdir -p /home/vagrant/.ssh
      echo "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKst079fkB+YnHa35U+Pa+jYyyHRi76hvdJ8p3wmxSA9 vagrant@admin" >> /home/vagrant/.ssh/authorized_keys
      chown -R vagrant:vagrant /home/vagrant/.ssh
      chmod 700 /home/vagrant/.ssh
      chmod 600 /home/vagrant/.ssh/authorized_keys
    SHELL

    m.vm.provider "vmware_desktop" do |v|
      v.vmx["displayname"] = "megaTP-node02"
      v.vmx["memsize"]     = "2048"
      v.vmx["numvcpus"]    = "2"
    end
  end

  config.vm.define "winsrv" do |m|
    m.vm.box = "StefanScherer/windows_2022"
    m.vm.hostname = "winsrv"
    m.vm.guest = :windows
    m.vm.communicator = "winrm"
    m.vm.network "private_network", ip: "192.168.77.13"

    m.winrm.username = "vagrant"
    m.winrm.password = "vagrant"
    m.winrm.retry_limit = 200
    m.winrm.retry_delay = 10
    m.winrm.timeout = 3600
    m.winrm.execution_time_limit = "PT2H"
    m.winrm.codepage = "65001"

    m.vm.provider "vmware_desktop" do |v|
      v.gui = true
      v.vmx["displayname"] = "megaTP-winsrv"
      v.vmx["memsize"]     = "4096"
      v.vmx["numvcpus"]    = "2"
    end
  end
end
