Unattended setup CentOS8 on KVM
===============================

This procedure will describe how to quickly setup CentOS8 with a static ip on a KVM host using kickstart.

Requirements
------------

This procedure requires a working kvm setup on Fedora32.

CentOS8 ISO
-----------

Download the latest CentOS8 iso and store it in /var/lib/libvirt/imagesredhat

Change the url to a mirror near you.
Find out which one here [CentOS mirrorlist](https://www.centos.org/download/)

Issue the following command

```bash
sudo curl -sLo /var/lib/libvirt/images/CentOS-8.2.2004-x86_64-dvd1.iso http://mirror.oxilion.nl/centos/8.2.2004/isos/x86_64/CentOS-8.2.2004-x86_64-dvd1.iso
```

Kickstart ISO
-------------

Generate The kickstart iso

```bash
sudo genisoimage -output /var/lib/libvirt/images/centos-ks.iso -volid OEMDRV -joliet -rock /path/to/ks.cfg
```

This kickstart configures centos8 with the following

* minimal install + python3
* fqdn: ipa01.linux.lab
* ip: 192.168.122.2
* netmask: 255.255.255.0
* gw: 192.168.122.1
* dns: 192.168.122.1
* root_passwd: redhat

KVM sda image
--------------

Create a sparse qcow2 image file on the command line for use by the linux guest.

As root issue the next command.

```bash
sudo qemu-img create -f qcow2 -o preallocation=metadata /var/lib/libvirt/images/ipa01_linux_lab_sda.qcow2 20G
```

Define KVM guest
----------------

As root issue the command

```bash
sudo virsh define --file /path/to/kvm_centos8.xml
```

This will result in a guest vm with 2 SATA cdroms attached.\

The first has the CentOS-8.2.2004-x86_64-dvd1.iso iso as source.\
The second has the centos-ks.iso as source.

Start guest vm
--------------

Start the vm and wait a couple minutes for it to finish.

```bash
sudo virsh start --domain ipa01.linux.lab
```

When you start the centos vm the installation media will wait in the boot menu for a short while.

You have the following options to choose from

* Do nothing\
  after a short while the iso will be checked\
  and the unattended install will proceed automatically after it is done.
* Press enter\
  after which the iso will be checked\
  and the unattended install will proceed automatically after it is done.
* Select the installation option and press enter\
  and the unattended install will start.

Post Installation
-----------------

All cdrom devices can be removed as they are no longer needed.
