# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "shekeriev/centos-8-minimal"

  config.vm.provider "virtualbox" do |vb|
  
    vb.name = "cephadm"
    vb.memory = "4096"
    vb.cpus = 4

    (0..2).each do |i|
      vb.customize [ "createmedium", "disk", "--filename", "osd-#{i}.vmdk", 
      "--format", "vmdk", "--size", 1024 * 10 ]
      vb.customize [ "storageattach", "cephadm" , "--storagectl", 
      "SATA", "--port", "#{i+1}", "--device", "0", "--type", 
      "hdd", "--medium", "osd-#{i}.vmdk", "--nonrotational", "on", "--discard", "on"]
    end
  end

  # Ceph Web GUI
  config.vm.network "forwarded_port", guest: 8443, host: 8443
  # Grafana
  config.vm.network "forwarded_port", guest: 3000, host: 3000

  config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.provision "shell", inline: <<-SHELL
    # Print commands and exit on error
    set -ex
    dnf install -y python3 podman
    curl --silent --remote-name --location https://github.com/ceph/ceph/raw/pacific/src/cephadm/cephadm
    chmod +x cephadm
    sudo ./cephadm add-repo --release pacific
    sudo ./cephadm install
    rm ./cephadm
    sudo cephadm bootstrap --mon-ip "$(hostname -I | cut -d' ' -f1)" --initial-dashboard-password "redhat1!" --single-host-defaults --dashboard-password-noupdate
    sudo cephadm shell -- ceph orch apply osd --all-available-devices
  SHELL

  $msg = <<MSG
------------------------------------------------------
Ceph deployment complete!

Access the Ceph Dashboard at:
https://localhost:8443

Access Grafana at:
https://localhost:3000

Log in as 'admin' with password 'redhat1!'

If you want cli access, use 'vagrant ssh', then 'sudo cephadm shell'

------------------------------------------------------
MSG

  config.vm.post_up_message = $msg

end
