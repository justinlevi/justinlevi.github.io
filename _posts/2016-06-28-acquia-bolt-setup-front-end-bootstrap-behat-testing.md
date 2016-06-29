---
layout: post
title: "Setting up Acquia Bolt"
subtitle: "Using Composer to manage Drupal, Bootstrap Sass custom theme, and behat testing"
bg-image: "post-bg.jpg"

date: 2016-06-28
---

An opinionated holistic approach to building Drupal projects.

Drupal is a lot more than just downloading and configuring modules. The ecosystem to build, test, and deploy is fragmented and confusing. In April this year, Acquia released an open source project called Bolt that was born out of Acquia's Professional Services team. This project is a collection of "best practices" from the folks who know the ins and outs of Drupal, and the ecosystem that Drupal operates in, better than anyone. If you are not aware of this project, you should be.

This talk will cover the following:

- Overview of what bolt is
- How to get setup
- Drupal-VM integration
- Phing and running the included ./blt.sh tasks
- Managing your D8 site with Composer
- Theming & front end development
- Behat & PHPUnit testing
- Deploying
- Continuous Integration w/ Travis

This talk is going to be less of a lecture and more of a demonstration. I'm going to be moving through a lot of this stuff pretty quickly but hopefully this gives everyone a high level overview of the technology.

Topics not covered, maybe next time:
- Features Integration
- Custom Module w/ Drupal console, building in Unit Testing
- Automated testing using live content
- Visual regression testing

Disclaimer: I'm definitely not an expert when it comes to Bolt. I'm a fan and I've invested a good chunk of time learning as much as I can. This talk is really just walking through what I've learned so far.

[x] Initial Setup video
[ ] Drupal installation
[x] Front-end setup
[ ] Deployment and Travis Video
[ ] Deploying to live site

##OVERVIEW##

***Problem Statement:***

As Drupal web designers and developers, we're faced with a ton of decisions as far as our work environment, testing, tools, best practices, and general strategies to get work done. This can create confusion and affect productivity.

***How this project helps:***

Leveraging Bolt takes the guess work out of trying to get many desperate technologies to all work in concert. Bolt can also help create a unified development platform for larger teams, which should improve productivity. Bolt has the potential to positively impact everything from onboarding new developers to continuous delivery of a product.

Teams can focus on the unique business impact of their project versus spending time on how the project architecture should look.

##HOW TO GET SETUP##

***Required Reading:***

- https://github.com/acquia/blt/blob/8.x/template/readme/onboarding.md
- https://github.com/acquia/blt/tree/8.x/template/readme
- https://github.com/acquia/blt/blob/8.x/template/tests/README.md

*Note, doing this on windows is possible but it is rough
I ended up going through this process from within another linux vm and then copying the files/folders back to my windows file system. I also needed to copy and manually modify the scripts/drupal-vm/config.yml and put it into my box folder. I had to copy the scripts/drupal-vm/Vagrantfile into the project root folder as well.  Trying to do this any other way caused lots of issues I couldn't figure out.*

From the onboarding readme:
```
We highly recommend that you do not use Windows directly for development. Many development tools (e.g., drush, gulp, etc.) are not built or tested for Windows compatibility. Furthermore, most CI solutions (e.g., Travis CI, Drupal CI, etc.) do not permit testing on Windows OS. Similarly, BLT cannot be fully tested on Windows and is unsupported on this platform.
```
**
If you're on windows, don't let this scare you though. There are ton of things you can learn here and you can get the project working.
**

**Requirements**
- Vagrant
- VirtualBox

###Initial Setup###

Clone the github.com/acquia/blt repository locally
$ `git clone github.com/acquia/blt`

From your terminal, cd into the cloned repository folder and run:
$ `./blt.sh clean && ./blt.sh configure`

This creates two files in the same folder,  `local.settings.php` and `project.yml`

Optional `project.yml` updates:

- update vendor_name, machine_name, human_name, profile > name, etc : for demonstration purposes I went with bolt
- update local > hostname : I use .vm because of some local dnsmasq setting conflicts
- create a new github repository on your account to store the project and update the `project.yml` git > remotes array
- Note: you may want to leave the lightning profile and thunder theme, otherwise the default behat tests won't run.

Create your project
$ `./blt.sh create`

This will create a new folder next to the repository directory that contains the entire project scaffolding

Note: drush aliases live in `drush/site-aliases/aliases.drushrc.php`. Update these once you're ready to deploy and to do some of the fancier database syncing etc.

###Project Setup###

$ `cd ../bolt`
// change directories to the project folder that was created depending on the name you chose

##DRUPAL-VM INTEGRATION##

Drupal-VM is a LAMP stack tuned for drupal development running on a Virtual Machine (VM). All that means is that it's an operating system (OS) running within your host OS. Bolt comes with drupal-vm out of the box, so getting things setup is pretty easy.

First, you need to have Vagrant and Virtual Machine downloaded and installed. Most of the time, the stock settings should work for you but if not, there are a ton of configuration options that you can checkout at the drupal-vm github repo. You can pretty easily turn on/off things like nodjs and ruby among many other useful libraries and packages. The documenation is great, so check it out.

http://docs.drupalvm.com/en/latest/

Initialize the Drupal-vm box folder and default config.yml
$ `./blt.sh vm:init`

**Installing extras**

Drupal-vm can install a bunch of libraries for you by updating the configuration file. The documentation on this is pretty lean within the bolt site so you need to go back to the Drupal-vm documentation to see what options you have.

Review the installed_extras section in the default.config.yml
https://github.com/geerlingguy/drupal-vm/blob/master/default.config.yml

The config.yml file that comes with Bolt will install some logical defaults for the project but in our case, we will want to add a few other things. You do that by copy/pasting the entire installed_extras yml array into the bolt `box/config.yml` file.

`box/config.yml`
```YAML
# Comment out any extra utilities you don't want to install. If you don't want
# to install *any* extras, make set this value to an empty set, e.g. `[]`.
installed_extras:
  - adminer
  # - blackfire
  - drupalconsole
  # - mailhog
  # - memcached
  # - newrelic
  - nodejs
  # - pimpmylog
  # - redis
  - ruby
  # - selenium
  # - solr
  # - varnish
  # - xdebug
  # - xhprof
```
- Enable adminer, drupalconsole, nodejs, and ruby by un-commenting those lines.
- Note: if you forget to do this before you do the initial $ vagrant up you can always run $ vagrant reload --provision  this will download and rebuild everything. If that fails, you can try $ vagrant halt && vagrant destroy && vagrant up
- Every now and then there will be a new release of the box. You will want to update using $ vagrant box update

Note: Drupal-VM ships with php7 which is great and you should switch if you can. However, Acquia cloud does not support php7 yet so if you need to work with php5.6, you need to change the following in your box/config.yml

`box/config.yml`
```YAML
# PHP Configuration. Currently-supported versions: 5.6, 7.0.
# To use 5.6, see: http://docs.drupalvm.com/en/latest/other/php-56/
# php_version: "7.0"
php_version: "5.6"
php_packages:
  - php5
  - php5-apcu
  - php5-mcrypt
  - php5-cli
  - php5-common
  - php5-curl
  - php5-dev
  - php5-fpm
  - php5-gd
  - php5-xsl
  - php5-sqlite
  - php-pear
  - libpcre3-dev
php_conf_paths:
  - /etc/php5/fpm
  - /etc/php5/apache2
  - /etc/php5/cli
php_extension_conf_paths:
  - /etc/php5/fpm/conf.d
  - /etc/php5/apache2/conf.d
  - /etc/php5/cli/conf.d
php_fpm_daemon: php5-fpm
php_fpm_conf_path: "/etc/php5/fpm"
php_fpm_pool_conf_path: "/etc/php5/fpm/pool.d/www.conf"
php_mysql_package: php5-mysql
php_memcached_package: php5-memcached

xhprof_download_url: https://github.com/phacility/xhprof/archive/master.tar.gz
xhprof_download_folder_name: xhprof-master
```
source: http://docs.drupalvm.com/en/latest/other/php-56/

Create your virtual machine
$ `vagrant up`

Connect to the VM via terminal
$ `vagrant ssh`
// This will create an ssh session to the virtual machine just like any hosting server

Change the working directory and run the setup task
$ `cd /var/www/bolt && ls -ahl`
// This will just verify that the folder syncing is setup and working

Logout of the ssh session
$ `logout`

##GOTCHAS##
Make sure to update: drush/site-aliases/drupal-vm.aliases.drushrc.php to the domain specified in your project.yml file.

To **NOT** install lightning profile by default:
set contrib: false under the profile in use in project.yml.
https://github.com/acquia/blt/issues/20#issuecomment-216983267

Update your hosts file
// Windows: `C:\Windows\System32\drivers\etc\hosts`
// Mac: `/private/etc/hosts`

`192.168.88.88 bolt.vm adminer.bolt.vm`

Install drupal, create local settings and local behat
$ `./blt.sh setup`

OPTIONAL
$ `./blt.sh blt:alias`
// After the rest of this script executes, you need to run $ source ~/.bashrc

You should now be able to run
$ `blt -l`

###SUCCESS!###
At this point you should be able to open your browser and load up http://bolt.vm

##Some cleanup.##

commit all of your changes to your github repository
$ `git branch develop && git add -A && git commit -m "First bolt commit."`
$ `git config --global --edit`
// set your name and email. This info will display on github to identify who submitted each commit

Add your remote github url
$ `git remote add origin git@github.com:justinlevi/bolt.git`
// swap out with your repo address

Push to master (we should probably push to develop and use git-flow feature branches)
$ `git push -u origin master`

Notes:
- the above commands will only work if you have your ssh key uploaded to github. Otherwise you can use the https that github provides, and enter your credentials when prompted.
- There is a lot of "funkiness" when jumping between Windows and Mac when trying to get the project up and running. For example, after doing the initial setup on windows, then cloning this repo locally on my mac,  running $ `./blt.sh local:setup`, I had to then make sure to `chmod -R +x scripts` in order to get the alias created. This would only be an issue when some people on your team are on macs while others are on windows.

##Phing and running the included `./blt.sh` tasks##

Phing is just a task runner, similar to grunt, or gulp but built on php instead of js. Phing uses xml to define tasks. I'm not exactly sure the rationale behind choosing phing over js for this project but my assumption has to do with the skillset that most drupal teams come with and the maturity of the project versus something like gulp.

So, we've already used a few phing tasks this far (./blt.sh clean, configure, setup).

Use the following command to get an overview of all of the available tasks.
$ `./blt.sh -l` or `blt -l` , configure, create, local:setup

There are a ton of commands here and not all of them will be applicable. The important thing to know is that many of the commands are built as building blocks and you can run them individually or as a complete block.

An example of this is the ./blt.sh deploy task which is comprised of several

Some commands worth knowing about:

- `./blt.sh local:sync` - grabs the remote database using drush aliases and synchronizes your local dev environment.
- `./blt.sh deploy` - useful for manually building and deploying to a remote git repo like what you find on Acquia (think hot-fix)
- `./blt.sh setup:behat` - creates a local.yml file for running behat tests locally
- `./blt.sh tests:behat` and all of the test commands...

##Managing your Drupal 8 Site w/ Composer##

What's interesting to note about the composer.json file that gets included with Bolt is that there are a number of dependencies set to fixed versions included out of the box. This is a non trivial list of "best practice" modules, libraries, etc. Just learning about what gets curated here is useful.

There are two main way to work with composer

1. Editing the `composer.json` file directly
2. Using the Command Line Interface (CLI)

Let's do both.

Let's add two drupal modules, Views and Admin Toolbar, the first we'll add by editing the composer file, the second we'll add via the cli.

Note: The `composer.json` repositories array defines the source for the modules and libraries you will include in the `require` and `require-dev` sections below. The official package repository is changing soon so the url for the composer type will need to be updated when that's ready.

*Adding Admin Toolbar to `composer.json` manually*

- Open up https://packagist.drupal-composer.org
- Do a search for your module, and click on the link
- The resulting page provides you with the cli command to add the module as well as the current version and how to reference this module in your composer.json (The ux is a bit confusing and could be more explicit for sure)
- Copy the title text "drupal/admin_toolbar" to the require array, as a key:value pair, the value being the version number
`"drupal/admin_toolbar": "8.1.15"`
- $ `composer update`

*Adding Views to `composer.json` w/ the cli*

- Open up a terminal window and cd to the project root folder
- $ `composer require drupal/adminimal_theme:8.1.1 --no-update`
- $ `composer update`

// Note, this is my preferred approach as it lets you know right away if you have something wrong in the syntax

##Theming and front-end development##

I'm a fan of bootstrap, mainly because it's what I know and I find it pretty easy to work with.

Install the bootstrap theme by adding the following to your composer.json
$ `"drupal/bootstrap":"8.3.0-beta3"`

Create a subtheme from one of the starter kits. Use the less version even though we'll be using the sass.
$ `cp -r docroot/themes/contrib/bootstrap/starterkits/less docroot/themes/custom/boltstrap`

Rename all of the THEMENAME.* files in your new custom subtheme

$ `mv THEMENAME.libraries.yml boltstrap.l ibraries.yml`
$ `mv THEMENAME.starterkit.yml boltstrap. yml`
$ `mv THEMENAME.theme boltstrap.theme`

Edit your THEME.yml file and change the name and anything else you want.

Install bower
$ `npm i -g bower`

Create a bower.json for the bootstrap sass project in the new custom theme
```JSON
{
  "name": "boltstrap",
  "version": "0.1.0",
  "authors": [],
  "description": "theme bower file",
  "license": "MIT",
  "ignore": [
  "**/.*",
  "node_modules",
  "bower_components",
  "tests"
  ],
  "dependencies": {
  "susy": "~2.2.5",
  "breakpoint-sass": "~2.6.1"
  },
  "devDependencies": {
  "bootstrap": "4.0.0-alpha.2"
  }
}
```

Install your bower dependencies
$ `bower install`

Create a package.json file with the following contents:
```JSON
{
  "name": "basetheme",
  "version": "0.1.0",
  "description": "baseline implementation of theme dependencies",
  "repository": {},
  "devDependencies": {
  "bower": "1.7.x",
  "browser-sync": "2.8.x",
  "del": "1.2.x",
  "eslint": "0.24.x",
  "eyeglass": "0.7.x",
  "gulp": "3.8.x",
  "gulp-autoprefixer": "3.1.x",
  "gulp-concat": "2.6.x",
  "gulp-css-globbing": "0.1.x",
  "gulp-sass-glob": "1.0.5",
  "gulp-debug": "2.1.x",
  "gulp-eslint": "1.0.x",
  "gulp-load-plugins": "0.10.x",
  "gulp-rename": "1.2.x",
  "gulp-sass": "2.0.x",
  "gulp-sass-lint": "1.0.x",
  "gulp-shell": "0.4.x",
  "gulp-size": "1.2.x",
  "gulp-sourcemaps": "1.5.x",
  "gulp-uglify": "1.4.x",
  "kss": "2.3.1",
  "node-sass": "3.4.x",
  "node-sass-import-once": "1.2.x",
  "run-sequence": "1.1.x",
  "sass-lint": "1.1.x",
  "stream-browserify": "2.0.x",
  "yargs": "3.29.0"
  },
  "engines": {
  "node": "0.12.x"
  },
  "private": true,
  "//": "The postinstall script is needed to work-around this Drupal core bug: https://www.drupal.org/node/2329453",
  "scripts": {
  "install-npm-bower" : "npm install && bower install;",
  "postinstall": "find node_modules/ -name '*.info' -type f -delete",
  "build": "gulp",
  "watch": "gulp watch"
  }
}
```

Install npm dependencies
$ `npm install`

Windows Gotcha: Scroll down below to read about "Creating a symbolic link on the VM". You will need to create a symlink to a node_modules folder outside of your shared NFS folder structure. You can create the node_modules folder in your vagrant home directory and the symlink can reside in your theme folder. This should allow you to install your npm dependencies.

Create a sass folder and add a styles.scss file
$ `mkdir sass && touch sass/styles.scss`

The following is a slightly modified version of the styles.scss that comes with bootstrap

/docroot/themes/custom/boltstrap/sass/styles.scss
```SASS
/*!
 * Bootstrap v4.0.0-alpha.2 (http://getbootstrap.com)
 * Copyright 2011-2015 Twitter, Inc.
 * Licensed under MIT (https://github.com/twbs/bootstrap/blob/master/LICENSE)
 */

// Core variables and mixins
@import "variables";
@import "../bower_components/bootstrap/scss/mixins";

// Reset and dependencies
@import "../bower_components/bootstrap/scss/normalize";
@import "../bower_components/bootstrap/scss/print";

// Core CSS
@import "../bower_components/bootstrap/scss/reboot";
@import "../bower_components/bootstrap/scss/type";
@import "../bower_components/bootstrap/scss/images";
@import "../bower_components/bootstrap/scss/code";
@import "../bower_components/bootstrap/scss/grid";
@import "../bower_components/bootstrap/scss/tables";
@import "../bower_components/bootstrap/scss/forms";
@import "../bower_components/bootstrap/scss/buttons";

// Components
@import "../bower_components/bootstrap/scss/animation";
@import "../bower_components/bootstrap/scss/dropdown";
@import "../bower_components/bootstrap/scss/button-group";
@import "../bower_components/bootstrap/scss/input-group";
@import "../bower_components/bootstrap/scss/custom-forms";
@import "../bower_components/bootstrap/scss/nav";
@import "../bower_components/bootstrap/scss/navbar";
@import "../bower_components/bootstrap/scss/card";
@import "../bower_components/bootstrap/scss/breadcrumb";
@import "../bower_components/bootstrap/scss/pagination";
@import "../bower_components/bootstrap/scss/pager";
@import "../bower_components/bootstrap/scss/labels";
@import "../bower_components/bootstrap/scss/jumbotron";
@import "../bower_components/bootstrap/scss/alert";
@import "../bower_components/bootstrap/scss/progress";
@import "../bower_components/bootstrap/scss/media";
@import "../bower_components/bootstrap/scss/list-group";
@import "../bower_components/bootstrap/scss/responsive-embed";
@import "../bower_components/bootstrap/scss/close";

// Components w/ JavaScript
@import "../bower_components/bootstrap/scss/modal";
@import "../bower_components/bootstrap/scss/tooltip";
@import "../bower_components/bootstrap/scss/popover";
@import "../bower_components/bootstrap/scss/carousel";

// Utility classes
@import "../bower_components/bootstrap/scss/utilities";
@import "../bower_components/bootstrap/scss/utilities-background";
@import "../bower_components/bootstrap/scss/utilities-spacing";
@import "../bower_components/bootstrap/scss/utilities-responsive";
```

Copy in the bootstrap provided default variables.scss
$ `cp ../bower_components/bootstrap/ scss/_variables.scss ./_variables.scss`

Delete the less folder so we're not confused later
$ `cd ../ && rm -rf less`

Create a gulpfile.js in your theme folder with the following contents:
```JS
'use strict';

var gulp = require('gulp'),
  sass = require('gulp-sass'),
  eslint = require('gulp-eslint'),
  notify = require('gulp-notify'),
  livereload = require('gulp-livereload'),
  sourcemaps = require('gulp-sourcemaps');

// Sass
gulp.task('styles', function() {
  return gulp.src(['sass/**/*.scss'])
  .pipe(sourcemaps.init())
  .pipe(sass({outputStyle: 'expanded'})
  .on('error', notify.onError(function (error) {
  return '\nAn error occurred while compiling sass.\n' + error.message;
  })
  )
  )
  .pipe(notify({
  title: 'SUCCESS!',
  message: 'Styles task complete' }))
  .pipe(sourcemaps.write('./'))
  .pipe(gulp.dest('css'));
});

// Lint Scripts Fails internally
gulp.task('lint', function () {
  return gulp.src('js/script.js')
  .pipe(eslint())
  .pipe(eslint.format())
  .pipe(eslint.results(function(results) {
  var count = results.errorCount;
  if (count > 0) {
  throw new Error('Failed with Errors');
  }
  }))
  .on('error', notify.onError(function (error) {
  return 'An error occurred while compiling JS.';
  }))
  .pipe(eslint.failAfterError());
});

// Scripts lint first and then success only if it doesn't fail
gulp.task('scripts', ['lint'], function () {
  return gulp.src('js/script.js')
  .pipe(notify({
  title: 'SUCCESS!',
  message: 'JS task complete' }));
});

// Watch
gulp.task('watch', function() {

  // Watch .scss files
  gulp.watch('sass/**/*.scss', ['styles']);

  // Watch .js files
  gulp.watch('js/**/*.js', ['scripts']);

  // Create LiveReload server
  livereload.listen();

  // Watch any files in dist/, reload on change
  gulp.watch(['css/**.css', '!css/**.css.map']).on('change', livereload.changed);

});

// The default task.
gulp.task('default', ['build']);

// #################
// Build everything.
// #################.
gulp.task('build', [
  'styles', 'scripts'
]);
```

Update your project.yml target-hooks > frontend-build
```YAML
target-hooks:
  # Executed when front end assets should be generated.
  frontend-build:
  # E.g., ${docroot}/sites/all/themes/custom/mytheme.
  dir: ${docroot}/themes/custom/boltstrap
  # E.g., `npm install` or `bower install`.
  command: npm build
  # Executed after deployment artifact is created.
```

This gives you the ability to now run
$ `./blt.sh frontend:build`

##BEHAT & PHPUNIT TESTING##

Well... mostly behat because phpunit gets kinda nutty and I ran out of time...

Connect to your VM
$ `vagrant ssh`
// We need to run our behat tests from within the VM

*GOTCHA* - (this isn't documented very well...)
$ `composer run-script install-phantomjs`

Check to make sure you have your tests/behat/local.yml file otherwise, run $ `./blt.sh setup:behat` // This will create a local.yml file. No other changes are necessary.

If your site is up and running with the steps above, you should be able to run:
$ `./blt.sh tests:behat`
// this will run 50 scenarios out of the gate. Note, on windows 10 of them failed for me.

Add the following to custom build.yml file.

build/custom/phing/build.yml
```YAML
behat:
  config: ${repo.root}/tests/behat/local.yml
  profile: local
  # If true, `drush runserver` will be used for executing tests.
  run-server: false
  # This is used for ad-hoc creation of a server via `drush runserver`.
  server-url: http://127.0.0.1:8888
  # If true, PhantomJS GhostDriver will be launched with Behat.
  launch-phantom: true
  # An array of paths with behat tests that should be executed.
  paths:
    # - ${docroot}/modules
    # - ${docroot}/profiles
    - ${repo.root}/tests/behat
  tags: '~ajax'
```
// This will allow us to only run our own tests in the /tests/behat folder using the ghost driver

Required reading:
https://github.com/acquia/blt/blob/8.x/template/tests/README.md

Troubleshooting
https://github.com/acquia/blt/issues/176#issuecomment-227288500

##PHPUNIT##

$ `blt tests:phpunit`
// This will create a reports folder which contains a nicely formatted html report.

Windows Gotcha: My first run I got an error on the testGitConfig saying that my `.git/hooks` was not yet created. This is a result of there not being a symbolic link to the hooks folder

On windows, creating a symbolic link on the VM is kind of a process.

Solving Windows `node_modules` nested directory issue
--------------
NOTE: FIRST MAKE SURE YOU DON'T HAVE YOUR VM RUNNING (As your regular user and admin)

Create/Edit `Vagrantfile.local` (same directory as the Vagrantfile - make sure to update the path to your folder
```BASH
config.vm.provider "virtualbox" do |vb|
  vb.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate//c/path/to/your/folder", "1"]
end
```
- Restart Cmder as administrator (Right click on the alias and choose run as administrator)

From a terminal on your host machine run:
$ `fsutil behavior set SymlinkEvaluation L2L:1 R2R:1 L2R:1 R2L:1`

Start up the VM and ssh back in
$ `cd C:\path\to\project && vagrant up && vagrant ssh`

From within a VM SSH session:
$ `cd /var/www/bolt && blt setup:git-hooks`

**HUGE GOTCHA** - exit all admin apps (commander and virtual box) before continuing

$ `logout`
$ `vagrant halt`

EXIT Cmder!!!
-----
 Note: If you don't exit out of your admin terminal you will get a UID error when trying to vagrant up again as your normal user.
-----


##BUILDING/DEPLOYING##

###Manually (should only need this for "hot fixes")###

Create the build artifact

$ `./blt.sh deploy:build`

// this will create a /deploy folder and compile the entire site ready for deployment. You can then manually commit and push the deploy folder to your remote repository

Alternatively you can run this "all-in-one" command that will build, commit, and attempt to deploy to the git remote repositories listed in your project.yml

$ `./blt.sh deploy -Ddeploy.branch=develop-build -Ddeploy.commitMsg='BLT-123: The commit message.'`
// this will download all dependencies with composer, commit, and push to a develop-build branch

###Automatically###

- Standard git process kicks off Travis which then pushes back to develop-deploy branch

Continuous Integration w/ Travis-CI

After you have created your repository for the project, head over to the travis-ci.org (if it's a public repository), travis-ci.com for private repositories if you have an account - $$$

Enable the repository by going to your profile page, clicking refresh ( I think the first time takes a minute or two)

Continuous integration is easier with a private account because you can add deploy keys. Doing any sort of automatic deployment from a public repository is a bit more complicated.

Here are the steps I took to make that happen:

- create a public key & private keys
- encrypt tjhe public key using th Travis command line tools
- commit this encrypted version to your repository
- add the the non-encrypted key to your github repository as a deploy key (repo settings > deploy keys)
- then load this key in your travis file with the command provided by the travis cli + wrapping in the following

```
if [[ "$TRAVIS_PULL_REQUEST" == "false" ]]; <<<TRAVIS CLI COMMAND HERE>>> -out ~/.ssh/id_rsa -d; chmod 600 ~/.ssh/id_rsa; ls -lash ~/.ssh; eval "$(ssh-agent -s)"; ssh-add ~/.ssh/id_rsa; fi
```

- you should be able to push to the develop branch and see a build get kicked off.

Resources:

Generating the keys
https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/

Instructions for encrypting a file with the travis cli
https://docs.travis-ci.com/user/encrypting-files/

Example of how this works in your travis file
https://github.com/acquia/blt/blob/8.x/.travis.yml#L27

Deploying to a live site.

- Clone the build branch to your hosting account, setup the database, etc.
- Add the following to the bottom of your .travis.yml

```
after_deploy:
  - yes | ssh -o "StrictHostKeyChecking no" justinlevi@justinlevi.com "cd ~/webapps/bolt && git pull"
```

UPDATE: SADLY I COULDN'T GET THIS TO WORK 😔
At this point I need to manually go to the site and do a git pull to deploy everything.

Keeping your BLT site up-to-date

$ `./blt.sh blt:update`

Sadly this does not work on Windows.
- Issue posted on the Acquia Bolt github repo. Seems to be an issue with Vagrant and/or NFS

Notes/LInks:

- Vagrant https://www.vagrantup.com
- Virtual Box https://www.virtualbox.org/wiki/Downloads
- Acquia Bolt https://github.com/acquia/blt
- Drupal VM http://www.drupalvm.com
- Behat  http://docs.behat.org/en/v3.0
- PHPUnit https://phpunit.de
- Phing https://www.phing.info

- Travis-CI Tips https://docs.travis-ci.com/user/deployment/heroku

Pro Tip

Install this plugin to speed up composer based installs on drupal-vm vagrant
-  `vagrant plugin install vagrant-cachier`
source: http://docs.drupalvm.com/en/latest/deployment/composer/#improving-composer-build-performance
