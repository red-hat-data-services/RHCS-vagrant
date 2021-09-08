# RHCS-vagrant

A simple way to demo the new cephadm-based Dashboard using Red Hat Ceph Storage 5 **downstream** bits with Vagrant and VirtualBox on RHEL 8.

## Dependencies

1. [Vagrant](https://www.vagrantup.com/)
1. [VirtualBox](https://www.virtualbox.org/) or libvirt

## Howto

1. Clone this repository  
   `git clone https://github.com/red-hat-data-services/RHCS-vagrant.git`
1. Change into the new directory  
   `cd RHCS-vagrant`
1. Run `vagrant up --provision`

This will ask you for your Red Hat Network credentials (these are the same as the ones you use on access.redhat.com).

After the provisioning is done (might take up to 30 minutes), you can log into the Dashboard at [https:localhost:8443](https:localhost:8443) or Grafana at [https:localhost:3000](https:localhost:3000)

To log in use: \
username : `admin` \
password : `redhat1!`

## Cleanup

To remove the Vagrant machine, use `vagrant destroy` while in the repository's folder. This will permanently remove the machine and all dependencies. Afterwards you can create a new machine with `vagrant up --provision`.
