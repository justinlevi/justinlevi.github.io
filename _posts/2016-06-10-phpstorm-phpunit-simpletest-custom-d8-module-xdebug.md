---
layout: post
title: "Debugging PHPUnit Simpletests in a custom D8 Module from within PHPStorm"
subtitle: "Struggling to get better at things through testing"
bg-image: "php-storm-phpunit.jpg"

date: 2016-06-02
---

I am struggling to debug PHPUnit Tests (with xdebug) in a custom D8 Module from PHPStorm or via the command line. Is it possible to do this?


**Here's what I have tried**
========================

*Command Line*
---

I can run my tests from within an ssh session on the VM with the following command:

$ `php core/scripts/run-tests.sh --url http://data.vm --color --verbose --module data_logger`

I have tried to initiate the debug session from the command line with the following:

$ `export XDEBUG_CONFIG="idekey=PHPSTORM remote_enable=1 remote_connect_back=0 remote_mode=req remote_port=9000";
sudo php core/scripts/run-tests.sh --url http://data.vm --color --verbose --module data_logger`

$ `sudo php -dxdebug.remote_enable=1 -dxdebug.remote_mode=req -dxdebug.remote_port=9000 -dxdebug.remote_host=192.168.88.1
-dxdebug.remote_connect_back=0 core/scripts/run-tests.sh --url http://data.vm --color --verbose --module data_logger`

Note, I don't think I should have to specify any of the xdebug variables on the command line because they are already set, but I was grasping at straws...

The tests execute, but the debugger session never seems to start within PHPStorm.


*PHP Storm*
---

PHPStorm **Miserably Fails** running my custom module PHPUnit Test.

PHPStorm **SUCCEEDS** running and debugging the Bolt included PHPUnit tests in the `tests/phpunit` folder.

After setting the PHPUnit settings screen to use a remote interpreter and specifying a custom autoloader to the root vendor folder `/var/www/data/vendor/autoload.php`


[![PHPStorm PHPUnit Remote Interpreter Settings][1]][1]

I right click and choose run/debug on one of the files, and everything works as expected. The debugger starts and we're good to go.

The command that is displayed in the console is:

`sftp://vagrant@192.168.88.88:22/usr/bin/php -dxdebug.remote_enable=1 -dxdebug.remote_mode=req -dxdebug.remote_port=9000 -dxdebug.remote_host=192.168.88.1 /home/vagrant/.phpstorm_helpers/phpunit.php --no-configuration Drupal\\Tests\\PHPUnit\\BuildTest /var/www/data/tests/phpunit/BuildTest.php`

The main difference that I see is that these tests extent `TestBase` where as my Test extends `WebTestBase` and the PHPUnit tests at the root level include a phpunit.xml file which I have tried to copy and update to point to the vendor/autoload.php as well but this didn't have any effect.

What I'm confused about is why PHPStorm seems to inherently understand that the `tests/phpunit/*` are PHPUnit tests when I right click on those files but when I right click on my Test class file, I get the option to debug with Javascript or as a PHP Script. Why does PHPStorm not recognize my test class as a PHPUnit test?

Bottom line, how can I debug a custom module test in PHPStorm?



Additional Context:
---

I'm running an [Acquia Bolt generated site][2] on a [Drupal-VM][3] Virtual Machine running PHP7 w/ XDebug Enabled. The site is using composer.json as the dependency manager for drupal core as well as all contrib modules. This is just a project for me to test some things, so here's the [github repo][4].

I can verify that xdebug is running on the VM as expected:

    $ php -i | grep xdebug
    /etc/php5/cli/conf.d/20-xdebug.ini,
    xdebug
    xdebug support => enabled
    xdebug.auto_trace => Off => Off
    xdebug.cli_color => 0 => 0
    xdebug.collect_assignments => Off => Off
    xdebug.collect_includes => On => On
    xdebug.collect_params => 0 => 0
    xdebug.collect_return => Off => Off
    xdebug.collect_vars => Off => Off
    xdebug.coverage_enable => On => On
    xdebug.default_enable => On => On
    xdebug.dump.COOKIE => no value => no value
    xdebug.dump.ENV => no value => no value
    xdebug.dump.FILES => no value => no value
    xdebug.dump.GET => no value => no value
    xdebug.dump.POST => no value => no value
    xdebug.dump.REQUEST => no value => no value
    xdebug.dump.SERVER => no value => no value
    xdebug.dump.SESSION => no value => no value
    xdebug.dump_globals => On => On
    xdebug.dump_once => On => On
    xdebug.dump_undefined => Off => Off
    xdebug.extended_info => On => On
    xdebug.file_link_format => no value => no value
    xdebug.force_display_errors => Off => Off
    xdebug.force_error_reporting => 0 => 0
    xdebug.halt_level => 0 => 0
    xdebug.idekey => PHPSTORM => PHPSTORM
    xdebug.max_nesting_level => 256 => 256
    xdebug.max_stack_frames => -1 => -1
    xdebug.overload_var_dump => On => On
    xdebug.profiler_aggregate => Off => Off
    xdebug.profiler_append => Off => Off
    xdebug.profiler_enable => Off => Off
    xdebug.profiler_enable_trigger => Off => Off
    xdebug.profiler_enable_trigger_value => no value => no value
    xdebug.profiler_output_dir => /tmp => /tmp
    xdebug.profiler_output_name => cachegrind.out.%p => cachegrind.out.%p
    xdebug.remote_addr_header => no value => no value
    xdebug.remote_autostart => Off => Off
    xdebug.remote_connect_back => On => On
    xdebug.remote_cookie_expire_time => 3600 => 3600
    xdebug.remote_enable => On => On
    xdebug.remote_handler => dbgp => dbgp
    xdebug.remote_host => localhost => localhost
    xdebug.remote_log => /tmp/xdebug.log => /tmp/xdebug.log
    xdebug.remote_mode => req => req
    xdebug.remote_port => 9000 => 9000
    xdebug.scream => Off => Off
    xdebug.show_error_trace => Off => Off
    xdebug.show_exception_trace => Off => Off
    xdebug.show_local_vars => Off => Off
    xdebug.show_mem_delta => Off => Off
    xdebug.trace_enable_trigger => Off => Off
    xdebug.trace_enable_trigger_value => no value => no value
    xdebug.trace_format => 0 => 0
    xdebug.trace_options => 0 => 0
    xdebug.trace_output_dir => /tmp => /tmp
    xdebug.trace_output_name => trace.%c => trace.%c
    xdebug.var_display_max_children => 128 => 128
    xdebug.var_display_max_data => 512 => 512
    xdebug.var_display_max_depth => 3 => 3


I can verify that my test displays and runs as expected at `admin/config/development/testing`.

[![Data Logger PHPUnit SimpleTest][5]][5]

[![enter image description here][6]][6]

Note: I can pause the test if I initiate the test through Drupal at `/admin/config/development/testing`. This is not an ideal workflow as it's slow and I'd rather not switch out of the IDE. I would rather initiate the test from within PHPStorm or from the command line.

Here are my PHP, Debug, and PBGp Proxy Settings as well:
[![PHP Storm - PHP Settings][7]][7]
[![PHP Storm - Debug Settings][8]][8]
[![PHP Storm - dbgp-proxy settings][9]][9]


  [1]: http://i.stack.imgur.com/AUdnK.jpg
  [2]: https://github.com/acquia/blt
  [3]: https://github.com/geerlingguy/drupal-vm
  [4]: https://github.com/justinlevi/data
  [5]: http://i.stack.imgur.com/YTqPD.jpg
  [6]: http://i.stack.imgur.com/fnhMH.jpg
  [7]: http://i.stack.imgur.com/4qqmb.jpg
  [8]: http://i.stack.imgur.com/cxphC.jpg
  [9]: http://i.stack.imgur.com/8qxhr.jpg
