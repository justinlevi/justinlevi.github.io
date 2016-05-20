---
layout: post
title: "The case of the failed PHPStorm Composer install"
subtitle: "Custom Private Repository on Github and SSH Keys"
bg-image: "burger-bg.jpg"

date: 2016-04-04
---

Here's a PHPStorm gotcha I ran into recently. 

I was seeing a permission denied issue when trying to run Composer Install from within a PHPStorm SSH session on my VM. This permissions issue was in relation to a custom module hosted on a private repository at Github. 

Come to find out, PHPStorm uses it's own built in SSH executable by default which didn't seem to be picking up my ssh keys for github. Switching PHPStorm to use the Native SSH executable for git seems to have fixed the glitch.

File > Settings > Version Control > Git

![PHPStorm windows settings version control git](/images/posts/051716/phpstorm-windows-settings-version-control-git.jpg){: .img-responsive}

Change SSH executable from "Built-in" to "Native"

Onward...