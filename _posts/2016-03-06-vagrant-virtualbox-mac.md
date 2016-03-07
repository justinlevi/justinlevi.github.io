---
layout: post
title: "Automating OSX Vagrant/VM on startup + Transmit Mounted Disks"
subtitle: "Something I did manually for way too long..."
bg-image: "post-bg.jpg"

date: 2016-03-06
---

For the last few months I've been really digging into my local development environment setup. Specifically  I've been using Vagrant and Virtual Box with the [Drupal-VM](https://github.com/geerlingguy/drupal-vm). Here's [some instructions I put together](https://github.com/justinlevi/drupal-vm-config) and my custom configs for Mac & Windows . I usually start my day by opening my terminal, doing a `cd` into the virtual machine directory and then doing `vagrant up`. I also realized recently that I could be using my VM as a mounted disk through Transmit. Sounds like an opportunity for automation...

### Where to start??

Well, first thing I needed to overcome was the password prompt that vagrant gives you when doing a $ `vagrant up`. Doing a bit of googling led me to this info:

```
sudo visudo Type your password, and you're editing the file. You'll want to paste these lines below (depending on whether you are running Vagrant on OS X or Linux.

As of version 1.7.3, the sudoers file in OS X should have these entries:
Cmnd_Alias VAGRANT_EXPORTS_ADD = /usr/bin/tee -a /etc/exports
Cmnd_Alias VAGRANT_NFSD = /sbin/nfsd restart
Cmnd_Alias VAGRANT_EXPORTS_REMOVE = /usr/bin/sed -E -e /*/ d -ibak /etc/exports
%admin ALL=(root) NOPASSWD: VAGRANT_EXPORTS_ADD, VAGRANT_NFSD, VAGRANT_EXPORTS_REMOVE
```
<sub>source: http://askubuntu.com/questions/412525/vagrant-up-and-annoying-nfs-password-asking</sub>

so, let's bang open our terminal and add those lines.

![alt text](/images/posts/030616/sudo-visudo.gif)
