BOXBUILDER
==========

Boxbuilder builds boxes!  Basic Bash scripts to bootstrap barebones-OS boxes with
[chef-solo](http://wiki.opscode.com/display/chef/Chef+Solo).  It can also create
EC2 AMIs.  Currently it only supports Debian, but pull requests with support for
new distros are welcome.

Quick Start
===========

To build a box, run the 'boxbuilder\_bootstrap' or 'boxbuilder-remote-bootstrap' scripts.
See details in the sections below.

General Usage Notes
===================

* All scripts are controlled via environment variables, which are documented below
* Environment variables can also be specified in ~/.boxbuilderrc
* If you set the 'boxbuilderrc\_url' environment variable to point to a remote file,
  it will be downloaded to ~/.boxbuilderrc\_downloaded, and a ~/.boxbuilderrc file
  will be automatically created with a line to read it.  This allows you to have
  a standard config always downloaded to ~/.boxbuilderrc\_downloaded, and add
  local overrides or customization in ~/.boxbuilderrc
* Environment variables and other config can be made via the 'boxbuilder\_config'
  variable.  This will be evaluated as a line of bash scripting AFTER
  ~/.boxbuilderrc is loaded.  This is useful to pass along configuration when using
  the 'remote' scripts to set up a remote box, or to easily override config in
  the ~/.boxbuilderrc from the command line when invoking the 'boxbuilder' script
  directly.


boxbuilder\_bootstrap script
===========================

'boxbuilder\_bootstrap' is a helper script which will download the boxbuilder
project to ~/.boxbuilder, and run the main 'boxbuilder' script.  It
is intended to be easily invoked on a clean box via
wget or curl with a bash one-liner.  For example, log in or SSH to the box you
wish to build, and paste the following:

    wget -O /tmp/boxbuilder_bootstrap http://github.com/thewoolleyman/boxbuilder/raw/master/boxbuilder_bootstrap && chmod +x /tmp/boxbuilder_bootstrap && /tmp/boxbuilder_bootstrap
    # If you want to poke around, log out or source ~/.bashrc after the first build

boxbuilder\_bootstrap environment variables
-------------------------------------------

    boxbuilder_repo=git://github.com/thewoolleyman/boxbuilder.git

The Git Repository from which to download and run the boxbuilder project.  If
the ~/.boxbuilder/boxbuilder file already exists, nothing will be downloaded
or overwritten.

    boxbuilder_config="export override_variable1=value; export override_variable2=value"

The contents of the 'boxbuilder\_config' variable should be Bash commands to export and override
boxbuilder config variables from their default values or values which were loaded from
~/.boxbuilderrc.  It will be evaluated by the 'boxbuilder\_bootstrap' script, which
means they will also be set when the 'boxbuilder' script is invoked by 'boxbuilder\_bootstrap'

boxbuilder\_remote\_bootstrap script
====================================

'boxbuilder\_remote\_bootstrap' allows you to run 'boxbuilder\_bootstrap' on a
remote box without logging in to it.  This makes it easy to hook boxbuilder into
some other automated process to automatically build boxes.

boxbuilder\_remote\_bootstrap environment variables
---------------------------------------------------

    boxbuilder_keypair=path_to_private_key
    boxbuilder_user=user_for_box_to_build
    boxbuilder_host=hostname_or_ip_of_box_to_build

Set 'boxbuilder\_keypair' to the path of your private key which will allow you to log in to
machine to build (you should already have your public key in ~/.ssh/authorized_keys).  Set 
'boxbuilder\_user' to the user, and 'boxbuilder\_host' to the hostname or IP address of the
machine you want to build.

'boxbuilder\_remote\_bootstrap' will issue remote SSH commands to automatically download
and run 'boxbuilder\_bootstrap'.

    boxbuilder_config="export override_variable1=value; export override_variable2=value"

The contents of the 'boxbuilder\_config' variable should be Bash commands to export and override
boxbuilder config variables from their default values or values which will be loaded from
~/.boxbuilderrc on the box build built.  It is NOT directly evaluated by the 'boxbuilder\_remote\_bootstrap'
script; but is only passed on to the 'boxbuilder\_bootstrap' script when it is invoked via SSH on
the remote box which is being built.

    boxbuilder_bootstrap_url=http://github.com/thewoolleyman/boxbuilder/raw/master/boxbuilder_bootstrap

'boxbuilder\_bootstrap\_url' is the location from which the boxbuilder\_bootstrap script will be downloaded
onto the remote box being built.  Override it to use your custom boxbuilder\_bootstrap script instead of
the default.

boxbuilder script
=================

'boxbuilder' is the main script which does the work of building a box (technically,
preparing a box to run your custom chef cookbooks).  When run 
on a barebones box with a newly installed OS, it will do the following:

* Install basic packages required for next steps
* Install Ruby via [RVM](http://rvm.beginrescueend.com/)
* Install [chef-solo](http://wiki.opscode.com/display/chef/Chef+Solo)
* Download Chef repositories you specify containing custom cookbooks, config, and JSON attributes.
* Create and run a script which executes chef-solo with the config and JSON attributes you specify

boxbuilder environment variables
--------------------------------

    boxbuilderrc_url=http://github.com/thewoolleyman/boxbuilder/raw/master/boxbuilderrc_example

'boxbuilderrc\_url' is the URL to a boxbuilder config script which will be automatically
downloaded to ~/.boxbuilderrc\_download on the box which is being built.

    boxbuilder_chef_repos=git://github.com/thewoolleyman/boxbuilder_chef_repo.git[,git://github.com/user/custom_chef_repo.git[,...]]

'boxbuilder\_chef_repos' is a comma-delimited list of Chef Git repositories which will be automatically
downloaded by boxbuilder.  They will be checked out under ~/.chef on the box which is being built.

    boxbuilder_chef_config_path=/home/user/.chef/boxbuilder_chef_repo/config/solo.rb

'boxbuilder\_chef\_config\_path' is the path to the Chef config file. boxbuilder will use
this as the '--config' parameter when it automatically creates and runs a chef-solo script on the
box which is being built.  This should be a path to a file
in one of your 'boxbuilder\_chef\_repos' which boxbuilder automatically downloaded to
~/.chef/{your repo}

    boxbuilder_chef_json_path=/home/user/.chef/boxbuilder_chef_repo/config/node.json

'boxbuilder\_chef\_json\_path' is the path to the Chef JSON attributes file. boxbuilder will use
this as the '--json-attributes' parameter when it automatically creates and runs a chef-solo script
on the box which is being built.  This should be a path to a file
in one of your 'boxbuilder\_chef\_repos' which boxbuilder automatically downloaded to
~/.chef/{your repo}


boxbuilder\_build\_ami script
=============================

TODO: docs

boxbuilder\_remote_build\_ami script
====================================

TODO: docs

TESTING
=======

Boxbuilder has a test suite which runs boxbuilder against a live machine, then asserts that it was
built correctly.  It requires the same SSH variables to be set as 'boxbuilder\_remote\_bootstrap'
('boxbuilder\_keypair', 'boxbuilder\_user', and 'boxbuilder\_host').  It will also read these and
any other variables set in ~/.boxbuilderrc.