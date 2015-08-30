---
layout:     post
title:      Vagrant Updates
date:       2015-07-04 20:47:19
author:     jrgriffiniii
summary:    Overcoming Challenges with Vagrant Boxes
categories: vagrant centos
thumbnail:  
tags:
 - site
---

After working leveraging Vagrant in the provisioning of several CentOS 6.x boxes over the past months, I've come across two incredibly tedious and persistent issues:

# Authentication and Packaging Your Boxes

When first attepting to ``vagrant package`` the state of the VM provisioned using one of various Puppet solutions, my attempts to initialize a new Vagrant environment using one of these Boxes locally has yielded forth the following output:

```
default: Inserting generated public key within guest...
default: Removing insecure key from the guest if its present...
default: Key inserted! Disconnecting and reconnecting using new SSH key...
default: Warning: Authentication failure. Retrying...
default: Warning: Authentication failure. Retrying...
```

[It appears that others have experienced this impediment within issue #5186](https://github.com/mitchellh/vagrant/issues/5186).

Fortunately, as outlined by various members of the community, there are several solutions which proved to be effective.  For me, the following steps were necessary before packaging a Box:

## Over SSH...
```
$ wget --no-check-certificate https://raw.githubusercontent.com/mitchellh/vagrant/master/keys/vagrant.pub \
-O ~/.ssh/authorized_keys
```
_(This adds the insecure Vagrant public key)._

## In the Vagrantfile...
```
  # Every Vagrant development environment requires a box. You can se
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "your-vagrant-box"
  config.ssh.insert_key = false # This must be overridden

  # Disable automatic box update checking. If you disable this, then
```
_(Now, Vagrant should not override authorized\_keys)._


*Please note here that this effectively renders your Box completely insecure; Hence, it must not be deployed into a production environment from this state*

I've typically found the need to update (``vagrant box update``) or remove (``vagrant box remove``) any previously-added versions of Boxes from one's local environment before these modifications resolved difficulties related to authentication.

# Network Devices and Packaging Your Boxes

Yet another plaguing difficulty which I've experienced lies with the management of system network devices for CentOS 6.x base Boxes.  Substantial discussion can be found within [issue #1777](https://github.com/mitchellh/vagrant/issues/1777).

I've only found success in outright removing the following file:

```
$ ls /etc/udev/rules.d/
[..] 70-persistent-cd.rules   70-persistent-net.rules  99-fuse.rules
$ sudo rm /etc/udev/rules.d/70-persistent-net.rules
$ ls /etc/udev/rules.d/
[..] 60-vboxadd.rules  70-persistent-cd.rules  99-fuse.rules
```

Please note for this solution, it was again necessary to either update or remove the packaged Box.  Further, please note that *this file is regenerated during the instantiation of every new VM instance by Vagrant*.

Hopefully this will prove to be of some utility to those who stumble across this post.  The best of luck with your efforts using Vagrant.
