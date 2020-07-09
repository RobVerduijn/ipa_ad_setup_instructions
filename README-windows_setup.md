Setup Windows 2019 server on KVM
================================

This procedure will generate a windows 2019 server install with a static ip config.

Requirements
------------

This procedure requires a working kvm setup on Fedora32.\
The virtio-win package must be installed.\
You must have an installation medium for the windows 2019 server.

Virtio-Win
----------

To use the virtio-win drivers the virtio-win repo has to be configured.

This repo configuration is valid for CentOS and Fedora.\
Issue the following command.

```bash
sudo curl -sLo /etc/yum.repos.d/virtio-win.repo https://fedorapeople.org/groups/virt/virtio-win/virtio-win.repo
```

Install the virtio-win package.

Issue the command

```bash
sudo dnf install -y virtio-win
```

* Stable (virtio-win-0.1.171-1) was used by me to test this procedure
* Do **NOT** use Latest (virtio-win-0.1.185-1)\
 It  will break the unattended install

To make the latest version easely accesible for the virt-manager tool
a symlink has to be created.\
As root link the image to the libvirt image pool.\
This way the latest stable version will always be available.

```bash
sudo ln -s /usr/share/virtio-win/virtio-win.iso /var/lib/libvirt/images/virtio-win.iso
```

KVM sda image
--------------

Create a sparse qcow2 image file on the command line for use by the windows guest.

Issue the next command.

```bash
sudo qemu-img create -f qcow2 -o preallocation=metadata /var/lib/libvirt/images/ad01_windows_lab_sda.qcow2 50G
```

Autounattend.xml
----------------

The following static ip configuration will be set up by the unattended install.

* no hostname will be configured (will be done by ansible later)
* ip address: 192.168.122.99
* ip mask: 255.255.255.0
* gateway: 192.168.122.1
* dns: 192.168.122.1
* Administrator_passwd: RedHat82

These are easely found in the Autounattended.xml, feel free to adjust them.

The unattended install will configure winrm so the system can be managed by ansible.\
See details here [Ansible Docs](https://docs.ansible.com/ansible/latest/user_guide/windows_setup.html#winrm-setup)\
The unattended install uses the following 2 commands:\
**Do not issue these commands**, the unattended install will do it for you

```powershell
powerShell Invoke-WebRequest -Uri http://raw.githubusercontent.com/ansible/ansible/devel/examples/scripts/ConfigureRemotingForAnsible.ps1 -OutFile $env:temp\ConfigureRemotingForAnsible.ps1

powershell -ExecutionPolicy ByPass -File C:\Users\Administrator\AppData\Local\Temp\ConfigureRemotingForAnsible.ps1 -DisableBasicAuth -EnableCredSSP -Verbose

```

The unattended install will set the password to ```RedHat82```\
This is visible in plain text in the autounattend file.\
Adjust this to your liking.

Edit the Autounattend.xml
-------------------------

Be very carefull when editing the xml, the WinPE installation tool is very unforgiving and it does not produce any usefull errors when it fails.\
So take care when editing the xml, and double check the final result with xml linting.

Find the key entry, read the comments and the instructions for kms keys on [Microsoft Docs](https://docs.microsoft.com/en-us/windows-server/get-started/kmsclientkeys)
Replace the fake key with a real one between the ```<Key>``` tags

```xml
<Key>XXXXX-XXXXX-XXXXX-XXXXX-XXXXX</Key>
```

Use the genisofs command to create an ISO image containing the Autounattend.xml file for easy windows 2019 server installation.\
WinPe requires the volid to be ```cidata```\
Anything else will cause the unattended install to fail.

```bash
sudo genisoimage -output /var/lib/libvirt/images/autounattend.iso -volid cidata -joliet -rock /path/to/Autounattend.xml
```

Plain Text Password
---

Regarding crypting the password in the Autounattend.xml file.\
The WinPE installation tool cannot deal with real encryption for passwords.\
Microsoft uses it's own base64 coding to encode the password.\
Which requires the use of a `null` character after each character in the password and padding it to a fixed size.\
This makes it really annoying to code them with base64 for use in the unattended install.\
However decoding it requires only ```echo "$base64_coded_password" | base64 -d```.\
So it adds zero security and lotsa frustration so don't bother with it.\
Keep it simple and make a password change mandatory after the installation is done.

Define KVM guest
----------------

As root issue the command

```bash
sudo virsh define --file /path/to/kvm_w2k19.xml
```

This will result in a guest vm with 3 SATA cdroms attached.\

The first has no source iso configured.\
The second has the virtio-win.iso as source.\
The third has the autounattend.iso as source.\

You will need to connect the first cdrom to your windows 2019 server installation media.

Windows 2019 Server installation media
--------------------------------------

Connect the windows 2019 installation iso.
Issue the following command:\
**edit the source to match your installation media location**

```bash
sudo virsh change-media --domain ad01.windows.lab --insert --path sdb --source /var/lib/libvirt/images/windows.iso
```

You will have to obtain the installation media for Windwos 2019 Server install.\
Ensure the installation media is present in the /var/lib/libvirt/images folder.

The media I used contained two versions of windows 2019 server

* Standard
* Datacenter

Each version allowed the installation of two profiles

* Core
* Desktop Experience

**The license Key determines if you get Standard or Datacenter.\
**You can see all these options if you boot from the media without an autounattend.iso attached.\
  If in doubt start the installation without the autounattend iso and klick through it,\
  the options will become visible before the actual installation starts.

The settings to toggle between these profiles can be found in the autounattend file between these tags

```xml
<InstallFrom>
    <MetaData wcm:action="add">
        <Key>/image/index</Key>
        <Value>2</Value>
    </MetaData>
</InstallFrom>
```

Changing the digit 2 to 1 will result in a core installation.

Core is the smallest version and contains a minimal set of tools and no GUI.\
Desktop experience is the desktop GUI that everybody is used to.\
The autounattend file configures the Desktop Experience.\
Core requires far less resources, it's faster and more secure due to the much smaller tool set.

All my configurations are done making use of ansible modules or powershell commands.\
So it does not matter which profile you choose.
However the Desktop Experience is more user friendly for the unexperienced user.

Start guest vm
--------------

After connecting the first cdrom to the windows installation media start the vm.

```bash
sudo virsh start --domain ad01.windows.lab
```

The installation will proceed and after quite some time you will see the desktop of the windows 2019 server.

Post Installation
-----------------

All cdrom devices can be removed as they are no longer needed.
