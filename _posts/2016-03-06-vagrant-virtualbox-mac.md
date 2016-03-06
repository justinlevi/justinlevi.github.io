---
layout: post
title: "Automating OSX Vagrant/VM on startup + Transmit Mounted Disks"
subtitle: "Something I did manually for way too long..."
bg-image: "post-bg.jpg"

date: 2016-03-06
---

For the last few months I've been really digging into my local development environment setup. Specifically  I've been using Vagrant and Virtual Box with the Drupal-VM. I usually start my day by opening my terminal, doing a `cd` into the virtual machine directory and then doing `vagrant up`. I also realized recently that I could be using my VM as a mounted disk through Transmit. Sounds like an opportunity for automation...

### Where to start??
