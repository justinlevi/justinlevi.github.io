---
layout: post
title: "Drupal 8 : twig_xdebug module tutorial"
subtitle: "Walkthrough for getting twig_xdebug setup and working in PHPStorm with an Acquia BLT Drupal 8 project"
bg-image: "monster-bg.jpg"

date: 2016-12-21
---

I recently started digging into some TWIG Drupal 8 development and needed to see what variables were available to me from within the twig template. You can sometimes use KINT or print the variables to the screen but this can be really slow and doesn't really give you the full picture. The twig_xdebug module lets you use your php debugger to inspect the variables you have access to, which can be really helpful.

### Get XDEBUG working

In your Drupal-VM folder, open the `config.yml` file and find the `installed_extras:` section.
Uncomment the `- xdebug` line.

Make the following changes to the xdebug configuration variables a bit further down in the same `config.yml` file.

```bash
# XDebug configuration. XDebug is disabled by default for better performance.
php_xdebug_default_enable: 1
php_xdebug_coverage_enable: 1
php_xdebug_cli_enable: 1
php_xdebug_remote_enable: 1
php_xdebug_remote_connect_back: 1
# Use PHPSTORM for PHPStorm, sublime.xdebug for Sublime Text.
php_xdebug_idekey: PHPSTORM
php_xdebug_max_nesting_level: 256
```

Save `config.yml` and then open the terminal app, navigate to the same folder that your config.yml file lives in, and run:
$ `vagrant reload --provision`

Verify that xdebug is up and running

Screengrab Video
<iframe width="750" height="500" src="https://www.youtube.com/embed/Nj6wmHUpJ9c" frameborder="0" allowfullscreen> </iframe>

In your terminal window, ssh into the Virtual Machine (VM)  with: $ `vagrant ssh` and run $ `php -i | grep xdebug`. If xdebug is running, you should see a bunch of lines related to xdebug on the screen.

The next step is to get PHPStorm setup to listen to xdebug breakpoints.

---
### Setup PHPStorm to listen for XDebug

First, make sure your VM is running - $ `vagrant up`.

Next, we will need to setup PHP to use the remote interpreter on your VM.  Open up the PHPStorm Preferences (⌘ + ,) and go to Languages & Frameworks > PHP. Click the "..." to the right of the CLI Interpreter field. Hint: type "CLI Interpreter" in the search field to find this quicker.

![PHP Remote Interpreter](/images/posts/122116/php-remote-interpreter.jpg){: .img-responsive}

Screengrab Video
<iframe width="750" height="500" src="https://www.youtube.com/embed/UobdcPiN_Us" frameborder="0" allowfullscreen> </iframe>

This will open up another modal window. Click on the plus icon, choose "Remote" and then select the "Vagrant" radio button and navigate to Vagrant Instance Folder. In my case it's the box folder for my project. PHPStorm will automatically fill in the remaining settings after which you can just click ok.

You may get an RSA warning that you can ignore and click yes. On the following screen, you can just click "ok".

Let's test to see that xdebug is running.

Install the chrome xdebug helper extension:
https://chrome.google.com/webstore/detail/xdebug-helper/eadndfjplgieldjbigjakmdgkmoaaaoc?hl=en

Back in PHPStorm, navigate to your index.php file and add a breakpoint somewhere by clicking in the gutter to the left of the code and to the right of the line numbers.

Start Listening for Debug Connections: Click the icon on the top that looks like a "old-timey" phone handset with a crossed out red circle at the top and little green bug at the bottom.

![PHPStorm Listen for xdebug connection](/images/posts/122116/phpstorm-listen-for-xdebug.jpg){: .img-responsive}

Jump back to chrome and navigate to your site's homepage. If all goes well, PHPStorm will listen for the xdebug connection and "catch" the request on the line you added the breakpoint to. You can then use your debugger to inspect the available php variables. To continue execution, you would press the green play button or to stop execution you would click the stop icon in the debugger panel.

Screengrab Video
<iframe width="750" height="500" src="https://www.youtube.com/embed/jHxIlD-Slwk" frameborder="0" allowfullscreen> </iframe>

---
### Install twig_xdebug

Back on your host machine, open a terminal window and navigate to your project root, where your composer.json file exists. In theory, you should just be able to run:

$ `composer require --dev drupal/twig_xdebug"

If you run into a php memory error like I did, you will need to increase the `memory_limit` in your php.ini file. The easiest way to do this is to first find which ini file is being loaded by running:

$ `php -i | grep  php.ini`

Look for the loaded Configuration file and open that up, search for "memory_limit" and increase the memory to at least 2048M. Save and run:

$ `php -i | grep memory_limit`

You should see the value you entered. You should now be able to run the composer require command above to install the module.

Screengrab Video
<iframe width="750" height="500" src="https://www.youtube.com/embed/Y7FE7SbCYnQ" frameborder="0" allowfullscreen> </iframe>

---
### Debug with twig_xdebug

Now that we have xdebug installed, we've verified it's working, and we have the twig_xdebug module installed, we next need to enable the module.

From within the VM, navigate to the docroot folder and run:

$ `drush en -y twig_xdebug` or you can enable it at `/admin/modules`

After that, we can add a "breakpoint" in the twig template. You do that by adding `{{ breakpoint() }}` to a twig file you know is loading.

Then, clear the registry via $ `drush cr`, switch back to chrome, make sure you have "Debug" selected in the chrome extension, and you are listening for xdebug connections from within PHPStorm and refresh the page.

If all goes well, PHPStorm should catch the breakpoint and you can then debug the variables available to the twig template. Note: I've noticed that PHPStorm doesn't always automatically get the focus so I just switch back and forth until I see the debug panel paused.

From her you can inspect all of the variables available to the template. Read the documentation on the twig_xdebug page for some additional information as well.

Screengrab Video

<iframe width="750" height="500" src="https://www.youtube.com/embed/-108L9nHaw4" frameborder="0" allowfullscreen> </iframe>

From the drupal.org/project/twig_xdebug page:

```
The breakpoint will open in a file (BreakpointExtension.php) outside your Twig template, but you'll be able to inspect any variables available at the breakpoint in the template. The key values you'll see at the breakpoint are:

$context: These are the variables available to use in the template.

$environment: This is information about the Twig environment, including available functions.

$arguments: If you supply an argument to breakpoint (e.g. {{ breakpoint(fields) }}), it'll be viewable here.
```


