---
layout: post
title: "Quick Look Essentials"
subtitle: "One of the more useful features on OS X"
bg-image: "chicken-run.jpg"

date: 2016-11-21
---

One of those things I missed the most when I had to switch back to a Windows machine at work was access to anything similar to Quick Look on OS X.

Hard to overstate how incredibly useful it is to be able quick look into source code files, including drupal extensions like .module and .inc. Viewing markdown with a spacebar click and the contents of a zip file, before unzipping, is really nice. 

Recently I upgraded to OS X Sierra and all of my Quick Look plugins disappeared. There are about a million different quick look plugins available out there but here's the list of plugins I've got running: 

- betterzipql (https://macitbetter.com/BetterZip-Quick-Look-Generator/)
- qlImageSize (https://github.com/Nyx0uf/qlImageSize)
- qlmarkdown (https://github.com/toland/qlmarkdown)
- qlcolorcode-extra (https://github.com/BrianGilbert/QLColorCode-extra)


You can install these with the following snippets if you have brew installed. 

```
brew install Caskroom/cask/betterzipql qlImageSize Caskroom/cask/qlmarkdown && wget https://github.com/BrianGilbert/QLColorCode-extra/archive/master.zip && unzip master.zip && mv QLColorCode-extra-master/QLColorCode.qlgenerator ~/Library/QuickLook/ && qlmanage -r
```

A couple of things that tripped me up. 

1. It's probably a good idea to reload quicklook after installing each of these. You can do that with $ `qlmanage -r`
2. `qlcolorcode` defaults to a theme that is no longer provided so you have to set the theme with the following command: 

$ `defaults write org.n8gray.QLColorCode hlTheme acid`

You can see some of the other themes available here (note - not all of them work): 
http://www.andre-simon.de/doku/highlight/en/highlight.php

I also wanted to see the line numbers and set the base font to `Monaco`. Here is the list of settings I'm using: 

```
defaults write org.n8gray.QLColorCode textEncoding UTF-16
defaults write org.n8gray.QLColorCode webkitTextEncoding UTF-16
defaults write org.n8gray.QLColorCode font Monaco
defaults write org.n8gray.QLColorCode extraHLFlags '-l -W'
defaults write org.n8gray.QLColorCode hlTheme acid
```

The above config looks like this: 

![QLColorCode](/images/posts/112116/qlcolorcode.jpg){: .img-responsive}

![QLImageSize](/images/posts/112116/qlimagesize.jpg){: .img-responsive}

![qlzip](/images/posts/112116/qlzip.jpg){: .img-responsive}






