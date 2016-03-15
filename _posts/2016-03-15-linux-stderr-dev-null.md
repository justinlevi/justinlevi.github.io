---
layout: post
title: "Shell commands ending in 2>/dev/null"
subtitle: "Getting drupal console auto completion to work."
bg-image: "drupal-console-auto-completion.jpg"

date: 2016-03-15
---

[From installing Drupal Console:](https://github.com/hechoendrupal/DrupalConsole#enabling-autocomplete)

---

5) Take advantage of command autocomplete feature.

#Bash or Zsh: Add this line to your shell configuration file:

source "$HOME/.console/console.rc" 2>/dev/null

---

But it wasn't working for me for some reason...

First, so, what's this all mean? Well, basically you're redirecting the application/command error output (stderr) to a black hole. In other words you don't want any errors printed to the screen.

( `>` ) = output redirection

( `2` ) = STDERR

**Standard In, out, and error**
Standard In is usually from the keyboard. Programs usually print to standard output and sometimes prints to standard error.
0, 1, 2 = STDIN, STDOUT, STDERR

source:
http://www.xaprb.com/blog/2006/06/06/what-does-devnull-21-mean

---

In order to get the autocompletion to work, I added this command to my ~/.bash_profile. In the screenshot gif below you can see the command at the bottom of my ~/.bash_profile - for some reason it wasn't working at the top of the file.

![alt text](/images/posts/031516/drupal-console-auto-complete.gif){: .img-responsive}
