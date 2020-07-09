ipa_ad_setup_instructions
=========================

This git repository contains a howto setup a kvm environment to test freeipa and ms-ad integration

This will be done through unattended installs.

KVM Host
--------

I have tested these instructions on my fedora32 desktop which I use as a kvm host.

The following packages/groups should be installed on the kvm host

* '@virt'
* genisoimage
* libguestfs-tools-c
* python3-libvirt

Ensure the libvirtd service is enabled and started.\
Ensure that the default virtual subnet 192.168.122.0/24 is running.\
If you have never modified the default virtual subnet, all should be fine.

Ansible
-------

Ansible will be used for the configuration of freeipa.\
The fedora32 desktop will be used as an ansible controller.

Install ansible packages.\
The version I used was 2.9.10

```bash
sudo dnf install -y ansible python3-ansible-lint ansible-freeipa sshpass
```

VM guests
---------

* Windows 2019 Server
* CentOS 8

CentOS 8 ipa server guest
-------------------------

For CentOS 8 setup see [CentOS8_setup](README-CentOS8_setup.md)

Windows 2019 Server
-------------------

For windows setup see [Windows_setup](README-windows_setup.md)

IPA and AD trust preparation
----------------------------

Run the play site.yml

```bash
ansible-playbook site.yml
```
