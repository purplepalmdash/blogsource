+++
categories = ["Virtualization"]
date = "2016-11-07T14:59:09+08:00"
description = "Turn Qcow2 into Vagrant box"
keywords = ["Virtualization"]
title = "TurnQcow2IntoVagrantBox"

+++
### img preparation
First you got a qcow2 file, which could let you startup a virtualmachine, so you
 startup a machine like following:    

```
# cp YourQcowFile.qcow2 ./box.img
# qemu-system-x86_64 -net nic -net user,hostfwd=tcp::2222-:22 -hda ./box.img -m 512
--enable-kvm
```
Login to the machine and run following scripts:    

```
useradd -m vagrant
mkdir /home/vagrant/.ssh
chmod 700 /home/vagrant/.ssh
echo "ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA6NF8iallvQVp22WDkTkyrtvp9eWW6A8YVr+kz4TjGYe7gHzIw+niNltGEFHzD8+v1I2YJ6oXevct1YeS0o9HZyN1Q9qgCgzUFtdOKLv6IedplqoPkcmF0aYet2PkEDo3MlTBckFXPITAMzF8dJSIFo9D8HfdOV0IAdx4O7PtixWKn5y2hMNG0zQPyUecp4pzC6kivAIhyfHilFR61RGL+GPXQ2MWZWFYbAGjyiYJnAmCP3NOTd0jMZEnDkbUvxhMmBYSdETk1rRgm+R4LOzFUGaHqHDLKLX+FIPKcF96hrucXzcWyLbIbEgE98OHlnVYCzRdK8jlqm8tehUc9c9WhQ== vagrant insecure public key" > /home/vagrant/.ssh/authorized_keys
chmod 600 /home/vagrant/.ssh/authorized_keys
chown -R vagrant.vagrant /home/vagrant/.ssh

# Sudoers
sed -i -r 's/(.* requiretty)/#\1/' /etc/sudoers

cat > /etc/sudoers.d/90-vagrant-users <<EOF
# User rules for vagrant
vagrant ALL=(ALL) NOPASSWD:ALL
EOF
```
Now close the machine.     

### Metadata file
Create a new metadata file like following:    

```
# vim metadata.json
{
  "provider"     : "libvirt",
  "format"       : "qcow2",
  "virtual_size" : 4
}
```
Create a Vagrantfile like following:    

```
$ vim Vagrantfile
# The contents below were provided by the Packer Vagrant post-processor

Vagrant.configure("2") do |config|
  config.vm.provider :libvirt do |libvirt|
    libvirt.driver = "kvm"
  end
end


# The contents below (if any) are custom contents provided by the
# Packer template during image build.
```
Now tar the box file via following:    

```
# tar czvf Trusty.box ./metadata.json ./Vagrantfile ./box.img 
```

### Using box
Add box file via:    

```
# vagrant box add Trusty.box --name "trusty64"
```
