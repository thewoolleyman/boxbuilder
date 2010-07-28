BOXBUILDER
==========

Boxbuilder builds boxes!  Basic Bash scripts to bootstrap barebones-OS boxes with
[chef-solo](http://wiki.opscode.com/display/chef/Chef+Solo).  It can also create
EC2 AMIs.  Currently it only supports Debian, but pull requests with support for
new distros are welcome.

Tracker Project: [http://www.pivotaltracker.com/projects/101913](http://www.pivotaltracker.com/projects/101913)

AMI-building code is based on Eric Hammond's tutorial at http://alestic.com/2010/01/ec2-ebs-boot-ubuntu

WARNING!  The 'build\_ami' scripts will automatically create EC2 resources for which you will be charged!
They automatically start and stop instances, but if the scripts fail or are killed
the instances may be left running.  Learn how to delete any unwanted resources via the the EC2 console
before using the 'build\_ami' scripts: [https://console.aws.amazon.com/ec2/home](https://console.aws.amazon.com/ec2/home)

----

Quick Start
===========

To build a box, download and run
'[boxbuilder\_bootstrap](http://github.com/thewoolleyman/boxbuilder/raw/master/boxbuilder_bootstrap)'
on a clean Debian box, or download and run
'[boxbuilder\_remote\_bootstrap](http://github.com/thewoolleyman/boxbuilder/raw/master/boxbuilder_remote_bootstrap)'
from your local shell.

To build an AMI image, download and run
'[boxbuilder\_build\_ami](http://github.com/thewoolleyman/boxbuilder/raw/master/boxbuilder_build_ami)' on a running AMI
instance, or download and run
'[boxbuilder\_remote_build\_ami](http://github.com/thewoolleyman/boxbuilder/raw/master/boxbuilder_remote_build_ami)'
from your local shell.

See details in the sections below.

----

General Usage Notes
===================

* All scripts are controlled via environment variables, which are documented below
* Environment variables can also be specified in ~/.boxbuilderrc
* If you set the 'boxbuilderrc\_url' environment variable to point to a remote file,
  it will be downloaded to ~/.boxbuilderrc\_download, and a ~/.boxbuilderrc file
  will be automatically created with a line to read it.  This allows you to have
  a standard config always downloaded to ~/.boxbuilderrc\_download, and add
  local overrides or customization in ~/.boxbuilderrc
* Environment variables and other config can be made via the 'boxbuilder\_config'
  variable.  This will be evaluated as a line of bash scripting AFTER
  ~/.boxbuilderrc is loaded.  This is useful to pass along configuration when using
  the 'remote' scripts to set up a remote box, or to easily override config in
  the ~/.boxbuilderrc from the command line when invoking the 'boxbuilder' script
  directly.

----

boxbuilder\_bootstrap script
===========================

'boxbuilder\_bootstrap' is a single downloadable helper script which will check out the
boxbuilder project to ~/.boxbuilder, and run the main '~/.boxbuilder/boxbuilder' script.  It
is intended to be easily invoked on a clean box via wget or curl with a bash one-liner.  
For example, log in or SSH to the box being built, and paste the following:

    wget -O /tmp/boxbuilder_bootstrap http://github.com/thewoolleyman/boxbuilder/raw/master/boxbuilder_bootstrap && chmod +x /tmp/boxbuilder_bootstrap && /tmp/boxbuilder_bootstrap
    # Be sure to log out or source ~/.bashrc after the first build

----

boxbuilder\_bootstrap environment variables
-------------------------------------------

'boxbuilder\_repo', 'boxbuilder\_branch', and 'boxbuilder\_dir' specify
the Git repository, branch, and directory to use when cloning and running the boxbuilder project.  If
the ~/.boxbuilder/ directory already exists, nothing will be downloaded or overwritten.

    boxbuilder_repo=git://github.com/thewoolleyman/boxbuilder.git
    boxbuilder_branch=master
    boxbuilder_dir=$HOME/.boxbuilder/

----

The contents of the 'boxbuilder\_config' variable should be Bash commands to export and override
boxbuilder config variables from their default values or values which were loaded from
~/.boxbuilderrc.  It will be evaluated by the 'boxbuilder\_bootstrap' script, which
means they will also be set when the 'boxbuilder' script is invoked by 'boxbuilder\_bootstrap'

    boxbuilder_config="export override_variable1=value; export override_variable2=value"

----

boxbuilder\_remote\_bootstrap script
====================================

'boxbuilder\_remote\_bootstrap' is run on your local box.  It allows you to run 'boxbuilder\_bootstrap'
on a remote box without logging in to it.  It issues remote SSH commands to automatically
download and run 'boxbuilder\_bootstrap' on the box being built.  This also makes it easy
to hook boxbuilder into other tools or processes to automatically build/update multiple boxes.

----

boxbuilder\_remote\_bootstrap environment variables
---------------------------------------------------

**(REQUIRED)** Set 'boxbuilder\_keypair' to the path of your private key which will allow you to log in to the
box being built (you should already have your public key in ~/.ssh/authorized_keys).  Set 
'boxbuilder\_user' to the user, and 'boxbuilder\_host' to the hostname or IP address of the
box being built.

    boxbuilder_keypair=path_to_private_key
    boxbuilder_user=user_for_box_to_build
    boxbuilder_host=hostname_or_ip_of_box_to_build

----

The contents of the 'boxbuilder\_config' variable should be Bash commands to export and override
boxbuilder config variables from their default values or values which will be loaded from
~/.boxbuilderrc on the box build built.  It is NOT directly evaluated by the 'boxbuilder\_remote\_bootstrap'
script; but is only passed on to the 'boxbuilder\_bootstrap' script when it is invoked via SSH on
the remote box which is being built.

    boxbuilder_config="export override_variable1=value; export override_variable2=value"

----

'boxbuilder\_bootstrap\_url' is the location from which the boxbuilder\_bootstrap script will be downloaded
onto the remote box being built.  Override it to use your custom boxbuilder\_bootstrap script instead of
the default.

    boxbuilder_bootstrap_url=http://github.com/thewoolleyman/boxbuilder/raw/master/boxbuilder_bootstrap

----

boxbuilder script
=================

'boxbuilder' is the main script which does the work of building a box - specifically,
preparing a box to run your custom chef cookbooks, downloading and running them.  When run 
on a barebones box with a newly installed OS, it will do the following:

* Install basic packages required for next steps (which you can override/augment with chef)
* Install Ruby via [RVM](http://rvm.beginrescueend.com/)
* Install [chef-solo](http://wiki.opscode.com/display/chef/Chef+Solo)
* Download Chef repositories you specify which contain custom cookbooks, config, and JSON attributes.
* Create and run a script which executes chef-solo with the config and JSON attributes you specify

----

boxbuilder environment variables
--------------------------------

'boxbuilderrc\_url' is the URL to a boxbuilder config script which will be automatically
downloaded to ~/.boxbuilderrc\_download on the box which is being built.

    boxbuilderrc_url=http://github.com/thewoolleyman/boxbuilder/raw/master/boxbuilderrc_download_default

----

'boxbuilder\_prerequisite\_packages' is a space-separated list of packages which will be installed
on the box being built.  By default, it is the minimal set of packages required for RVM and Ruby
on a clean Debian install.  Other packages which will be used to other applications on the box
should be installed via chef, as part of the application setup cookbooks.  On Debian, you can
specify an exact version with {packagename}={packageversion}.  See the install_packages()
function in the boxbuilder script for the latest default packages.

    boxbuilder_prerequisite_packages={See the install_packages() function in the boxbuilder script for the latest default packages}

----

'boxbuilder\_rvm\_version' is the version of RVM to use.  By default, it will be the latest stable
version, listed at [http://rvm.beginrescueend.com/releases/stable-version.txt](http://rvm.beginrescueend.com/releases/stable-version.txt).

    boxbuilder_rvm_version=

----

'boxbuilder\_default\_ruby' is the version of the Ruby interpreter which will be installed as the
RVM default, and used to install and run chef.

    boxbuilder_default_ruby=$(curl -s http://rvm.beginrescueend.com/releases/stable-version.txt)

----

'boxbuilder\_chef\_dir' is the directory under which all chef-related files will be downloaded
and created.

    boxbuilder_prerequisite_packages={See the install_packages() function in the boxbuilder script for the latest default packages}

----

**(REQUIRED)** 'boxbuilder\_chef_repos' is a space-delimited list of Chef Git repositories which will be automatically
downloaded by boxbuilder.  They will be checked out under $boxbuilder\_chef\_dir (~/.chef) on the box which is being built.

    boxbuilder_chef_repos=git://github.com/thewoolleyman/boxbuilder_example1_chef_repo.git[ git://github.com/thewoolleyman/boxbuilder_example2_chef_repo.git[ ...]]

----

**(REQUIRED)** 'boxbuilder\_chef\_config\_path' is the path of the Chef config file. boxbuilder will use
this as the '--config' parameter when it automatically creates and runs a chef-solo script on the
box which is being built.  This should be a path to a file
in one of your 'boxbuilder\_chef\_repos'.

    boxbuilder_chef_config_path=$boxbuilder_chef_dir/boxbuilder_chef_repo/config/solo.rb

----

**(REQUIRED)** 'boxbuilder\_chef\_json\_path' is the path of the Chef JSON attributes file. boxbuilder will use
this as the '--json-attributes' parameter when it automatically creates and runs a chef-solo script
on the box which is being built.  This should be a path to a file
in one of your 'boxbuilder\_chef\_repos'.

    boxbuilder_chef_json_path=$boxbuilder_chef_dir/boxbuilder_chef_repo/config/node.json

----

'boxbuilder\_chef\_gem\_install\_options' allows you to pass custom options when installing the chef gem. This
can be used to install the gem with a different version, source, etc.

    boxbuilder_chef_gem_install_options="--no-ri --no-rdoc"

----

boxbuilder\_build\_ami script
=============================

'boxbuilder\_build\_ami' creates an EC2 EBS-backed AMI when run from an EC2 instance.  It does the following:

* Creates a chroot jail
* Executes the 'boxbuilder\_bootstrap' script while re-rooted within the chroot jail
* Creates and publishes an EBS-backed AMI from the chroot jail

----

boxbuilder\_build_ami environment variables
-------------------------------------------

The contents of the 'boxbuilder\_config' variable should be Bash commands to export and override
boxbuilder config variables from their default values or values which will be loaded from
~/.boxbuilderrc when 'boxbuilder\_bootstrap' is run in the chroot jail.
It IS directly evaluated by the 'boxbuilder\_remote\_bootstrap'
script; and is also passed on to the 'boxbuilder\_bootstrap' script when it is invoked via SSH on
the remote box which is being built.

    boxbuilder_config="export override_variable1=value; export override_variable2=value"

----

**(REQUIRED)** 'boxbuilder\_xxx\_yyy' zzz.

    boxbuilder_xxx=yyy

----

TODO: document these boxbuilder\_build\_ami variables:

    # EC2 Credentials
    boxbuilder_ec2_privatekey=${boxbuilder_privatekey:?"Please set 'boxbuilder_ec2_privatekey' to the path of your private key (See X.509 Certificates at https://aws-portal.amazon.com/gp/aws/developer/account/index.html?action=access-key#access_credentials)"}
    boxbuilder_ec2_cert=${boxbuilder_ec2_cert:?"Please set 'boxbuilder_ec2_cert' to the path of your cert (See X.509 Certificates at https://aws-portal.amazon.com/gp/aws/developer/account/index.html?action=access-key#access_credentials)"}
    boxbuilder_ec2_keypair=${boxbuilder_ec2_keypair:?"Please set 'boxbuilder_ec2_keypair' to the path of your keypair private key (https://console.aws.amazon.com/ec2/home#c=EC2&s=KeyPairs)"}
    boxbuilder_ec2_keypairname=${boxbuilder_ec2_keypairname:?"Please set "boxbuilder_ec2_keypairname" to the name of your keypair (https://console.aws.amazon.com/ec2/home#c=EC2&s=KeyPairs)"}

    # AMI Builder Settings
    boxbuilder_ami_instancetype=${boxbuilder_ami_instancetype:?"Please set 'boxbuilder_ami_instancetype' to the type of instance you want, e.g. m1.small for 32 bit and m1.large for 64 bit (See http://aws.amazon.com/ec2/instance-types/)"}
    boxbuilder_ami_prefix=${boxbuilder_ami_prefix:?"Please set 'boxbuilder_ami_prefix' to a string with no spaces.  This string will be prepended to the name of your new AMI"}



----

boxbuilder\_remote\_build\_ami script
=====================================

'boxbuilder\_remote\_build\_ami' is run from your local box.  It allows you to run 'boxbuilder\_build\_ami'
on a remote box without logging in to it.  It issues remote SSH commands to do the following:

* Automatically start an EC2 instance using your EC2 account and credentials
* Upload your credentials to the instance
* Download and run 'boxbuilder\_remote\_build\_ami' on the instance

----

boxbuilder\_remote\_build\_ami environment variables
----------------------------------------------------

**(REQUIRED)** 'boxbuilder\_xxx\_yyy' zzz.

    boxbuilder_xxx=yyy

----

EC2 Info
========

* Current Base AMI: ami-4b4ba522 - Ubuntu 10.04 LTS (Lucid Lynx) (us-east-1)

* Alestic AMI list: http://alestic.com/
* Ubuntu AMI list: 



Testing
=======

Boxbuilder has a test suite which runs boxbuilder against a live machine, then asserts that it was
built correctly.  It requires the same SSH variables to be set as does the 'boxbuilder\_remote\_bootstrap'
('boxbuilder\_keypair', 'boxbuilder\_user', and 'boxbuilder\_host').  It will also read these and
any other variables set in ~/.boxbuilderrc.