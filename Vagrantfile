# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "generic/rhel8"

  config.vm.provider :virtualbox do |vb|
  
    vb.name = "cephadm"
    vb.memory = "4096"
    vb.cpus = 4

    vb.customize ["storagectl", "cephadm", "--name", "SATA", "--add", "sata", "--controller", "IntelAhci"]

    (0..2).each do |i|
      vb.customize [ "createmedium", "disk", "--filename", "osd-#{i}.vmdk", 
      "--format", "vmdk", "--size", 1024 * 10 ]
      vb.customize [ "storageattach", "cephadm" , "--storagectl", 
      "SATA", "--port", "#{i+1}", "--device", "0", "--type", 
      "hdd", "--medium", "osd-#{i}.vmdk", "--nonrotational", "on", "--discard", "on"]
    end
  end

  config.vm.provider :libvirt do |lv|

    lv.cpus = 4
    lv.memory = "4096"

    (0..2).each do |i|
      lv.storage :file, :size => '10G'
    end
  end

  # Ceph Web GUI
  config.vm.network "forwarded_port", guest: 8443, host: 8443
  # Grafana
  config.vm.network "forwarded_port", guest: 3000, host: 3000

  config.vm.synced_folder ".", "/vagrant", disabled: true

  if Dir.glob("#{File.dirname(__FILE__)}/.vagrant/machines/default/*").empty? || ARGV[1] == '--provision'
    print "Please insert your Red Hat Network credentials\n"
    print "These are the same credentials you use on access.redhat.com\n"
    print "Username: "
    username = STDIN.gets.chomp
    print "Password (hidden): "
    password = STDIN.noecho(&:gets).chomp
    print "\n"

    config.vm.provision "shell", sensitive: true, env: {"USER" => username, "PASSWORD" => password}, inline: <<-SHELL
      # Print commands and exit on error
      set -ex
      subscription-manager register --username $USER --password $PASSWORD --force
      POOLID=$(subscription-manager list --matches 'Red Hat Ceph Storage' --available --pool-only | head -n1)
      subscription-manager attach --pool=$POOLID
      subscription-manager repos --disable='*'
      subscription-manager repos --enable=rhel-8-for-x86_64-baseos-rpms --enable=rhel-8-for-x86_64-appstream-rpms --enable=rhceph-5-tools-for-rhel-8-x86_64-rpms # --enable=ansible-2.9-for-rhel-8-x86_64-rpms
      dnf install -y cephadm
      cephadm bootstrap --mon-ip "$(hostname -I | cut -d' ' -f1)" --registry-url registry.redhat.io --registry-username $USER --registry-password $PASSWORD --initial-dashboard-password "redhat1!" --dashboard-password-noupdate --allow-fqdn-hostname
      cephadm shell -- ceph orch apply osd --all-available-devices
    SHELL
end

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
