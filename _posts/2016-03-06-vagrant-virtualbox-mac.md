---
layout: post
title: "Automating OSX Vagrant/VM on startup + Transmit Mounted Disks"
subtitle: "Something I did manually for way too long..."
bg-image: "post-bg.jpg"

date: 2016-03-06
---

For the last few months I've been really digging into my local development environment setup. Specifically  I've been using Vagrant and Virtual Box with the [Drupal-VM](https://github.com/geerlingguy/drupal-vm). Here's [some instructions I put together](https://github.com/justinlevi/drupal-vm-config) and my custom configs for Mac & Windows . I usually start my day by opening my terminal, doing a `cd` into the virtual machine directory and then doing `vagrant up`. I also realized recently that I could be using my VM as a mounted disk through Transmit. Sounds like an opportunity for automation...

### Where to start?? Let's get rid of the vagrant password requirement related to NFS

Well, first thing, if we want to automatically start up the VM when the system boots, we need to get rid of the password prompt that vagrant gives you when doing a $ `vagrant up`. Doing a bit of googling led me to this info:

---
sudo visudo Type your password, and you're editing the file. You'll want to paste these lines below (depending on whether you are running Vagrant on OS X or Linux.

As of version 1.7.3, the sudoers file in OS X should have these entries:

```bash
Cmnd_Alias VAGRANT_EXPORTS_ADD = /usr/bin/tee -a /etc/exports
Cmnd_Alias VAGRANT_NFSD = /sbin/nfsd restart
Cmnd_Alias VAGRANT_EXPORTS_REMOVE = /usr/bin/sed -E -e /*/ d -ibak /etc/exports
%admin ALL=(root) NOPASSWD: VAGRANT_EXPORTS_ADD, VAGRANT_NFSD, VAGRANT_EXPORTS_REMOVE
```
<sub>source:  https://www.vagrantup.com/docs/synced-folders/nfs.html</sub>

---
so, let's bang open our terminal and add those lines.

![alt text](/images/posts/030616/sudo-visudo.gif){: .img-responsive}

At this point we can `cd` into the directory where our VagrantFile lives and then do a $ `vagrant up`. We shouldn't have to enter our password. Yay!

### launchd.plist - Running `vagrant up` at startup

Ok, next step is to actually get run the $ `vagrant up` command when the OS boots up. This is a task for the launchd.plist so just to get a refresher on what we can do there, open up terminal and enter `man launchd.plist` or visit the [Apple Developer command reference](https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man5/launchd.plist.5.html). Per the instructions, and the example at the end, we need to create a plist file. Something like this should work:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>EnvironmentVariables</key>
	<dict>
		<key>PATH</key>
		<string>/usr/local/opt/php56/bin:/usr/local/bin:/usr/local/sbin:/usr/bin:/bin:/usr/sbin:/sbin:/Applications/DevDesktop/drush:/Applications/DevDesktop/drush</string>
	</dict>
	<key>Label</key>
	<string>com.wintercreative.vagrantup</string>
	<key>ProgramArguments</key>
	<array>
		<string>/usr/local/bin/vagrant</string>
		<string>up</string>
	</array>
	<key>RunAtLoad</key>
	<true/>
	<key>StandardErrorPath</key>
	<string>/tmp/com.wintercreative.vagrantup.err</string>
	<key>StandardOutPath</key>
	<string>/tmp/com.wintercreative.vagrantup.out</string>
	<key>WorkingDirectory</key>
	<string>/Users/justinwinter/Sites/drupal-vm</string>
</dict>
</plist>
```

I ended up using Sublime Text 3 to get this started and then opened it up in XCode because I like the plist editor there better.

Name this file `com.wintercreative.vagrantup.plist` and drop it in the `~/Library/LaunchAgents/` directory. Then you'll need to change the ownership and permissions of that file to the user you're logging in as.

```bash
$sudo chown justinwinter:staff ~/Library/LaunchAgents/com.wintercreative.vagrantup.plist
$sudo chmod 644 ~/Library/LaunchAgents/com.wintercreative.vagrantup.plist
```
Next you'll need to run launchctl to add the launch script to be loaded.
```
$sudo launchctl load -w ~/Library/LaunchAgents/com.wintercreative.vagrantup.plist ```

A few gotchas I ran into:

* I initially tried to drop this plist in the /Library/LaunchDaemons folder. This turned out to be a bad idea for a few reasons. Without getting into too much detail, this blog post is really incredible as far as breaking down the difference between the two. http://www.grivet-tools.com/blog/2014/launchdaemons-vs-launchagents Bottom line is that it's better to use ~/Library/LaunchAgents for this task.
* I use brew to manage a lot of my external apps and libraries. Using brew, vagrant gets installed in /usr/local/bing/vagrant and I had that path wrong to start. The [error comes up as 78](http://stackoverflow.com/questions/34215527) if you do a $`sudo launchctl list`.
* I thought I could get away without defining my PATH variables in the plist file despite seeing this in a [] stackoverflow post].(http://stackoverflow.com/questions/30680861)
* Finally, per another [stackoverflow post](http://stackoverflow.com/questions/34215527) I found a super helpful little app called **LaunchControl** which can be installed view brew as well $ `brew cask install launchcontrol`. http://www.soma-zone.com/LaunchControl/ - This totally helped me push past the last roadblock.
Ok, let's test it out one final time. From within your VM folder, run $ `vagrant halt` to make sure the VM isn't running, and then restart.

### OK. Final Piece. Mounting the server on startup as a disk using Transmit.

First, you have to download and install the [TransmitDisk](https://library.panic.com/transmit/td-install/)
   Transmit Disk allows you to mount any of your Favorites in the Finder itself, even if Transmit’s not running. These volumes are real: drag files to your SFTP server, save a small graphic to your Amazon S3 bucket directly from Photoshop, or roll your own backup volume.


Next I just [followed these instructions](https://library.panic.com/transmit/td-login-mount/)

```applescript
tell application "Transmit"
    set myFave to item 1 of (favorites whose name is "My Favorite")

    tell current tab of (make new document at end)
        connect to myFave with mount
        close
    end tell
end tell
```

I want this to run after vagrant is finished and not right at login because the VM hasn't finished provisioning yet and therefore can't be mounted. Task for another time...
