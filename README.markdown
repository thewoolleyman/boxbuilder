BOXBUILDER
==========

Boxbuilder builds boxes!  Basic Bash scripts to bootstrap barebones-OS boxes with
[chef-solo](http://wiki.opscode.com/display/chef/Chef+Solo).  It can also create
EC2 AMIs.  Currently it only supports Debian, but pull requests with support for
new distros are welcome.

Tracker Project: [http://www.pivotaltracker.com/projects/101913](http://www.pivotaltracker.com/projects/101913)

AMI-building code is based on Eric Hammond's tutorial at http://alestic.com/2010/01/ec2-ebs-boot-ubuntu

**WARNING!  The 'build\_ami' scripts will automatically create EC2 resources for which you will be charged!
They automatically start and stop instances and create EBS volumes, but if the scripts fail or are killed
the might not be removed. It is YOUR RESPONSIBILITY to ensure that any EC2 resources which boxbuilder creates are
automatically shut down.  If you do not YOU WILL BE CHARGED BY AMAZON UNTIL THEY ARE SHUT DOWN.
Learn how to delete any unused resources via the the EC2 console
before using the 'build\_ami' scripts: [https://console.aws.amazon.com/ec2/home](https://console.aws.amazon.com/ec2/home)**



----
&nbsp;


_Quick Start and Overview_
=========================

To build a box, download and run
'[boxbuilder\_bootstrap](http://github.com/thewoolleyman/boxbuilder/raw/master/boxbuilder_bootstrap)'
on a clean Debian box, or download and run
'[boxbuilder\_remote\_bootstrap](http://github.com/thewoolleyman/boxbuilder/raw/master/boxbuilder_remote_bootstrap)'
from your local shell.

To build an AMI image (of a 'box' built by boxbuilder in a chroot jail on an EC2 box), download and run
'[boxbuilder\_build\_ami](http://github.com/thewoolleyman/boxbuilder/raw/master/boxbuilder_build_ami)' on a running AMI
instance, or download and run
'[boxbuilder\_remote_build\_ami](http://github.com/thewoolleyman/boxbuilder/raw/master/boxbuilder_remote_build_ami)'
from your local shell.

You will be prompted to enter all required variables for the script you are running, such as
your EC2 credentials, SSH keys, locations of your Chef repositories, etc.  See details in the sections below.

Here's the invocation order of the scripts for the two main tasks (building a box or building an AMI).
You can, of course, run and re-run any of these scripts directly by logging into the box being built or the
EC2 image building the AMI.

Building a Box: 

1. 'boxbuilder\_remote\_bootstrap' (invoked from any Bash shell) issues SSH commands on the box being built to download and invoke...
2. 'boxbuilder\_bootstrap' (on the box being built), which clones a boxbuilder git repo, and invokes...
3. 'boxbuilder' (on the box being built) from the cloned repo.

Building an AMI Image: 

1. 'boxbuilder\_remote\_build\_ami' (invoked from any Bash shell) starts an EC2 instance, and issues SSH commands on it built to download and invoke...
2. 'boxbuilder\_build\_ami' (on the EC2 instance), which creates a chroot jail, and runs commands in the chroot jail to download and invoke...
3. 'boxbuilder\_bootstrap' (in the chroot jail), which clones a boxbuilder git repo, and invokes...
4. 'boxbuilder' (in the chroot jail) to build a 'box' in the chroot jail, which exits and returns control to...
5. 'boxbuilder\_bootstrap' (in the chroot jail), which exits and returns control to...
6. 'boxbuilder\_build\_ami' (on the EC2 instance), which continues to create an AMI image of the chroot jail.



----
&nbsp;


_Flexible, Automatically Downloaded, Easily Overridable Configuration_
======================================================================

* The boxbuilder script itself and the other scripts which invoke it locally or
  remotely are controlled via configuration values set in environment variables.
  Some have overridable defaults.  Others are required, and you will be prompted for
  them if they are not set.
* Environment variables can also be specified in the config file ~/.boxbuilderrc.  
  If it does not already exist, ~/.boxbuilderrc will automatically be created
  to read and source ~/.boxbuilderrc\_download.
* ~/.boxbuilderrc\_download will automatically be downloaded from the URL specified
  in 'boxbuilderrc\_url' environment variable.  By default, it points to
  the '[boxbuilderrc\_download\_default](http://github.com/thewoolleyman/boxbuilder/raw/master/boxbuilder_download_default)'
  file at [http://github.com/thewoolleyman/boxbuilder/raw/master/boxbuilder_download_default](http://github.com/thewoolleyman/boxbuilder/raw/master/boxbuilder_download_default)
* This simple config-file-based approach allows you to have standard config files for different
  machines stored in source control and always downloaded to ~/.boxbuilderrc\_download,
  but still add local overrides to the bottom of ~/.boxbuilderrc after it sources
  ~/.boxbuilderrc\_download if you want to test or debug by re-running the 'boxbuilder'
  script locally on the box being built.  If you prefer, you can create a
  ~/.boxbuilderrc file which which does not source ~/.boxbuilderrc\_download, and
  instead set all variables directly in ~/.boxbuilderrc, via 'boxbuilder\_config' (see next bullet)
  or in some other way you choose.
* Environment variables can also be set or overridden via the 'boxbuilder\_config'
  variable.  The contents of this variable will be evaluated as a line of bash scripting
  AFTER the ~/.boxbuilderrc and ~/.boxbuilderrc\_download config files are sourced.
  This provides another easy command-line-based approach to override environment variables
  when you are using the 'remote' scripts to set up a remote box or the
  'boxbuilder' or 'boxbuilder\_bootstrap' scripts directly.

Here's an ordered summary of the process for reading configuration from various sources
when the 'boxbuilder' script runs.  You can add/override environment variables at any point:

0. Any pre-existing variables are, naturally, already set before 'boxbuilder' is invoked
1. ~/.boxbuilderrc\_download is automatically downloaded
2. ~/.boxbuilderrc is created by default, with one line to source ~/.boxbuilderrc\_download
3. You can add overrides to the bottom of ~/.boxbuilderrc
4. ~/.boxbuilderrc is sourced by the 'boxbuilder' script
5. The contents of 'boxbuilder\_config' are evaluated.
6. Any environment variables not set by this point will either have a default value set, or will
   cause the script(s) to exit with a prompt if they are required.

----
&nbsp;


_boxbuilder\_bootstrap script_
==============================

'boxbuilder\_bootstrap' is a single downloadable helper script which will check out the
boxbuilder project to ~/.boxbuilder, and run the main '~/.boxbuilder/boxbuilder' script.  It
is intended to be easily invoked on a clean box via wget or curl with a bash one-liner.

For example, log in or SSH to the box being built, and paste the following:

    wget -O /tmp/boxbuilder_bootstrap http://github.com/thewoolleyman/boxbuilder/raw/master/boxbuilder_bootstrap && chmod +x /tmp/boxbuilder_bootstrap && /tmp/boxbuilder_bootstrap
    # Be sure to log out or source ~/.bashrc after the first build

----

boxbuilder\_bootstrap environment variables
-------------------------------------------

The 'boxbuilder\_repo', 'boxbuilder\_branch', and 'boxbuilder\_dir' are intended to make
it easy to use and hack on your fork of boxbuilder.  They let you specify
the Git repository, branch, and directory to use when cloning and running the boxbuilder project.  If
the 'boxbuilder\_dir' directory has already been cloned, nothing will be downloaded or overwritten, but
'git pull' will automatically be run with 'boxbuilder\_repo' and 'boxbuilder\_branch'.

    boxbuilder_repo=git://github.com/thewoolleyman/boxbuilder.git
    boxbuilder_branch=master
    boxbuilder_dir=$HOME/.boxbuilder/

----

The contents of the 'boxbuilder\_config' variable should be Bash commands to export and override
boxbuilder config variables from their default values or values which were loaded from
~/.boxbuilderrc.  They will be evaluated by the 'boxbuilder\_bootstrap' script, which
means they are also available when the 'boxbuilder' script is sourced by 'boxbuilder\_bootstrap'

    boxbuilder_config="export override_variable1=value; export override_variable2=value"

----
&nbsp;


_boxbuilder\_remote\_bootstrap script_
======================================

'boxbuilder\_remote\_bootstrap' is run from any Bash shell.  It allows you to run 'boxbuilder\_bootstrap'
on a remote box without logging in to it.  It issues remote SSH commands to automatically
download and run 'boxbuilder\_bootstrap' on the box being built.  This also makes it easy
to hook boxbuilder into other tools or processes to automatically build/update multiple boxes.

For example, run the following from a Bash shell on your workstation.  You will be prompted to set required variables
which you can export from your shell, or set in ~/.boxbuilderrc:

    wget -O /tmp/boxbuilder_remote_bootstrap http://github.com/thewoolleyman/boxbuilder/raw/master/boxbuilder_remote_bootstrap && chmod +x /tmp/boxbuilder_remote_bootstrap && /tmp/boxbuilder_remote_bootstrap


----

boxbuilder\_remote\_bootstrap environment variables
---------------------------------------------------

**(REQUIRED to log into the remote box being built)** Set 'boxbuilder\_keypair' to
the path of your private key which will allow you to log in to the
box being built (you should already have your public key in ~/.ssh/authorized_keys).  Set 
'boxbuilder\_user' to the user, and 'boxbuilder\_host' to the hostname or IP address of the
box being built.

    boxbuilder_keypair=path_to_private_key
    boxbuilder_user=user_for_box_to_build
    boxbuilder_host=hostname_or_ip_of_box_to_build

----

The contents of the 'boxbuilder\_config' variable should be Bash commands to export and override
boxbuilder config variables from their default values or values which will be loaded from
~/.boxbuilderrc on the box build built.  It will be evaluated by the 'boxbuilder\_remote\_bootstrap'
script; AND also passed on to the 'boxbuilder\_bootstrap' script when it is invoked via SSH on
the remote box which is being built.

    boxbuilder_config="export override_variable1=value; export override_variable2=value"

----

'boxbuilder\_bootstrap\_url' is the location from which the boxbuilder\_bootstrap script will be downloaded
onto the remote box being built.  Override it to use your custom boxbuilder\_bootstrap script instead of
the default.

    boxbuilder_bootstrap_url=http://github.com/thewoolleyman/boxbuilder/raw/master/boxbuilder_bootstrap

----
&nbsp;


_boxbuilder script_
===================

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

    boxbuilder_rvm_version=$(curl -s http://rvm.beginrescueend.com/releases/stable-version.txt)

----

'boxbuilder\_default\_ruby' is the version of the Ruby interpreter which will be installed as the
RVM default, and used to install and run chef.

    boxbuilder_default_ruby=$(curl -s http://rvm.beginrescueend.com/releases/stable-version.txt)

----

'boxbuilder\_chef\_dir' is the directory under which all chef-related files will be downloaded
and created.

    boxbuilder_prerequisite_packages={See the install_packages() function in the boxbuilder script for the latest default packages}

----

**(REQUIRED to specify your chef repos)** 'boxbuilder\_chef_repos' is a space-delimited list of Chef Git repositories which will be automatically
downloaded by boxbuilder.  They will be checked out under $boxbuilder\_chef\_dir (~/.chef) on the box which is being built.

    boxbuilder_chef_repos=git://github.com/thewoolleyman/boxbuilder_example1_chef_repo.git[ git://github.com/thewoolleyman/boxbuilder_example2_chef_repo.git[ ...]]

----

**(REQUIRED to specify your chef config)** 'boxbuilder\_chef\_config\_path' is the path of the Chef config file. boxbuilder will use
this as the '--config' parameter when it automatically creates and runs a chef-solo script on the
box which is being built.  This should be a path to a file
in one of your 'boxbuilder\_chef\_repos'.

    boxbuilder_chef_config_path=$boxbuilder_chef_dir/boxbuilder_chef_repo/config/solo.rb

----

**(REQUIRED to specify your chef config)** 'boxbuilder\_chef\_json\_path' is the path of the Chef JSON attributes file. boxbuilder will use
this as the '--json-attributes' parameter when it automatically creates and runs a chef-solo script
on the box which is being built.  This should be a path to a file
in one of your 'boxbuilder\_chef\_repos'.

    boxbuilder_chef_json_path=$boxbuilder_chef_dir/boxbuilder_chef_repo/config/node.json

----

'boxbuilder\_chef\_gem\_install\_options' allows you to pass custom options when installing the chef gem. This
can be used to install the gem with a different version, source, etc.

    boxbuilder_chef_gem_install_options="--no-ri --no-rdoc"

----
&nbsp;


_boxbuilder\_build\_ami script_
===============================

'boxbuilder\_build\_ami' creates an EC2 EBS-backed AMI when run from an EC2 instance.  It does the following:

* Creates a chroot jail
* Executes the 'boxbuilder\_bootstrap' script while re-rooted within the chroot jail (which builds your image, see documentation for 'boxbuilder\_bootstrap')
* Creates and publishes an EBS-backed AMI from the image which was created in the chroot jail

----

boxbuilder\_build_ami environment variables
-------------------------------------------

The contents of the 'boxbuilder\_config' variable should be Bash commands to export and override
boxbuilder config variables.  It is directly evaluated by the 'boxbuilder\_build\_ami'
script; and is also passed on to be evaluated before 'boxbuilder\_bootstrap' is run in the chroot jail.
See the documentation of the config options and the other scripts for more details on 'boxbuilder\_config'.

    boxbuilder_config="export override_variable1=value; export override_variable2=value"

----

**(REQUIRED by EC2 tools)** 'EC2\_CERT' is the path to your EC2 X509 certificate file.  It is directly required by the EC2 tools
executables (which is why it is not named 'boxbuilder\_...) to create and manage EC2 resources.  If it is not set, it will default 
to the first file matching $HOME/.ec2/cert-*.pem.
See [http://docs.amazonwebservices.com/AWSSecurityCredentials/1.0/AboutAWSCredentials.html#X509Credentials](http://docs.amazonwebservices.com/AWSSecurityCredentials/1.0/AboutAWSCredentials.html#X509Credentials) for documentation,
and [https://aws-portal.amazon.com/gp/aws/developer/account/index.html?ie=UTF8&action=access-key#access_credentials](https://aws-portal.amazon.com/gp/aws/developer/account/index.html?ie=UTF8&action=access-key#access_credentials) to create a cert and private key.

    EC2_CERT=$HOME/.ec2/cert-*.pem

----

**(REQUIRED by EC2 tools)** 'EC2\_PRIVATE\_KEY' is the path to your EC2 X509 private key file.  It is directly required by the EC2 tools
executables (which is why it is not named 'boxbuilder\_...) to create and manage EC2 resources.  If it is not set, it will default
to the first file matching $HOME/.ec2/pk-*.pem.
See [http://docs.amazonwebservices.com/AWSSecurityCredentials/1.0/AboutAWSCredentials.html#X509Credentials](http://docs.amazonwebservices.com/AWSSecurityCredentials/1.0/AboutAWSCredentials.html#X509Credentials) for documentation,
and [https://aws-portal.amazon.com/gp/aws/developer/account/index.html?ie=UTF8&action=access-key#access_credentials](https://aws-portal.amazon.com/gp/aws/developer/account/index.html?ie=UTF8&action=access-key#access_credentials) to create a cert and private key.

    EC2_PRIVATE_KEY=$HOME/.ec2/pk-*.pem

----

**(REQUIRED)** 'boxbuilder\_ami\_prefix' is a string which will be prepended to the name of the AMI being created.

    boxbuilder_ami_prefix="boxbuilder_test_$(date +%Y%m%d-%H%M)"

----

'boxbuilder\_bootstrap\_url' is the location from which the boxbuilder\_bootstrap script will be downloaded
into the AMI chroot jail being built.  Override it to use your custom boxbuilder\_bootstrap script instead of
the default.

    boxbuilder_bootstrap_url=http://github.com/thewoolleyman/boxbuilder/raw/master/boxbuilder_bootstrap

----
&nbsp;


_boxbuilder\_remote\_build\_ami script_
=======================================

**WARNING: It is YOUR RESPONSIBILITY to ensure that any EC2 resources which boxbuilder creates are
automatically shut down.  If you do not YOU WILL BE CHARGED BY AMAZON UNTIL THEY ARE SHUT DOWN.  See
the detailed warning at the top of this README.**

'boxbuilder\_remote\_build\_ami' is run from any bash shell.  It allows you to run 'boxbuilder\_build\_ami'
on a remote box without logging in to it:

* Download the Amazon EC2 API tools to your local filesystem
* Automatically start an EC2 instance (called the 'builder instance') using your EC2 account and credentials (which you are required to obtain and specify)
* Upload your credentials to the builder instance
* Issue remote SSH commands to download and run 'boxbuilder\_remote\_build\_ami' on the builder instance (which then builds your AMI, see documentation for 'boxbuilder\_build\_ami')

----

The contents of the 'boxbuilder\_config' variable should be Bash commands to export and override
boxbuilder config variables.  It is directly evaluated by the 'boxbuilder\_remote\_build\_ami'
script; and is also passed on to be evaluated before 'boxbuilder\_build\_ami' is run in on the
remote EC2 builder instance.  See the documentation of the config options and other scripts for more details on 'boxbuilder\_config'.

    boxbuilder_config="export override_variable1=value; export override_variable2=value"

----

**(REQUIRED by EC2 tools)** 'EC2\_CERT' is the path to your EC2 X509 certificate file.  It is directly required by the EC2 tools
executables (which is why it is not named 'boxbuilder\_...) to create and manage EC2 resources.  If it is not set, it will default 
to the first file matching $HOME/.ec2/cert-*.pem.
See [http://docs.amazonwebservices.com/AWSSecurityCredentials/1.0/AboutAWSCredentials.html#X509Credentials](http://docs.amazonwebservices.com/AWSSecurityCredentials/1.0/AboutAWSCredentials.html#X509Credentials) for documentation,
and [https://aws-portal.amazon.com/gp/aws/developer/account/index.html?ie=UTF8&action=access-key#access_credentials](https://aws-portal.amazon.com/gp/aws/developer/account/index.html?ie=UTF8&action=access-key#access_credentials) to create a cert and private key.

    EC2_CERT=$HOME/.ec2/cert-*.pem

----

**(REQUIRED by EC2 tools)** 'EC2\_PRIVATE\_KEY' is the path to your EC2 X509 private key file.  It is directly required by the EC2 tools
executables (which is why it is not named 'boxbuilder\_...) to create and manage EC2 resources.  If it is not set, it will default
to the first file matching $HOME/.ec2/pk-*.pem.
See [http://docs.amazonwebservices.com/AWSSecurityCredentials/1.0/AboutAWSCredentials.html#X509Credentials](http://docs.amazonwebservices.com/AWSSecurityCredentials/1.0/AboutAWSCredentials.html#X509Credentials) for documentation,
and [https://aws-portal.amazon.com/gp/aws/developer/account/index.html?ie=UTF8&action=access-key#access_credentials](https://aws-portal.amazon.com/gp/aws/developer/account/index.html?ie=UTF8&action=access-key#access_credentials) to create a cert and private key.

    EC2_PRIVATE_KEY=$HOME/.ec2/pk-*.pem

----

**(REQUIRED for EC2 tools)** 'EC2\_KEYPAIR' is the path to your EC2 'keypair' file.  Amazon refers to this as a 'keypair', but it is
actually just the private key.  You don't ever need to directly use the public key; Amazon manages it for you - but you can always
generate it using the '-y' option of 'ssh-keygen'.  'EC2\_KEYPAIR' is used by
'boxbuilder\_remote\_build\_ami' for SSH access to the EC2 builder instance it creates to run 'boxbuilder\_build\_ami'.
If it is not set, it will default to the first file matching $HOME/.ec2/keypair-*.pem.
NOTE: I don't think this variable is ever used directly by the EC2 tools, but it is named like the
other EC2-required credential variables for consistency.
See [http://docs.amazonwebservices.com/AWSSecurityCredentials/1.0/AboutAWSCredentials.html#EC2KeyPairs](http://docs.amazonwebservices.com/AWSSecurityCredentials/1.0/AboutAWSCredentials.html#EC2KeyPairs) for documentation,
and [https://console.aws.amazon.com/ec2/home#c=EC2&s=KeyPairs](https://console.aws.amazon.com/ec2/home#c=EC2&s=KeyPairs) to create a keypair.

    EC2_KEYPAIR=$HOME/.ec2/keypair-*.pem

----

**(REQUIRED for EC2 tools)** 'EC2\_KEYPAIR\_NAME' is the name of your EC2 'keypair'.  It is used by 'boxbuilder\_remote\_build\_ami'
when starting the EC2 builder instance it creates to run 'boxbuilder\_build\_ami'. You must set this to the
name which matches the keypair file specified in 'EC2\_KEYPAIR'.
NOTE: I don't think this variable is ever used directly by the EC2 tools, but it is named like the
other EC2-required credential variables for consistency.
See [http://docs.amazonwebservices.com/AWSSecurityCredentials/1.0/AboutAWSCredentials.html#EC2KeyPairs](http://docs.amazonwebservices.com/AWSSecurityCredentials/1.0/AboutAWSCredentials.html#EC2KeyPairs) for documentation,
and [https://console.aws.amazon.com/ec2/home#c=EC2&s=KeyPairs](https://console.aws.amazon.com/ec2/home#c=EC2&s=KeyPairs) to create a keypair.

    EC2_KEYPAIR_NAME=$HOME/.ec2/keypair-*.pem

----

'boxbuilder\_builder\_instance\_instance\_type' is the type of EC2 builder instance you wish to use or start.
It must match the type of AMI you wish to build (32-bit or 64-bit).  Use 'm1.small' if you are building a 32-bit AMI, and 
'm1.large' if you are building a 64-bit AMI.  The default value is 'm1.large' (64-bit).
See [http://aws.amazon.com/ec2/instance-types](http://aws.amazon.com/ec2/instance-types)

    boxbuilder_builder_instance_instance_type=m1.large

----

'boxbuilder\_builder\_instance\_ami\_id' is the AMI which you wish to use to start the EC2 builder instance.
It must match the type of AMI you wish to build (32-bit or 64-bit).  See the **'EC2 Info'** section below for
current 32-bit and 64-bit AMI IDs.  The default value is 'ami-4b4ba522' (64-bit).

    boxbuilder_builder_instance_ami_id=ami-4b4ba522

----

'boxbuilder\_user' is the username which is used to log into the EC2 builder instance via SSH.
It will default to 'ubuntu', which is the default user on the standard Ubuntu EC2 AMIs used to
start a builder instance.

    boxbuilder_user=ubuntu

----

'boxbuilder\_builder\_instance\_host' is the hostname of the EC2 builder instance.  If it is not set,
an instance will automatically be created for you.  Boxbuilder will ATTEMPT to terminate the instance after
the AMI is built, but this is not guaranteed.

**WARNING: If you set 'boxbuilder\_builder\_instance\_host' to one of your own preexisting EC2 instances
and do NOT set 'boxbuilder\_terminate\_builder\_instance' to false, the instance 
WILL BE TERMINATED and you will LOSE ANY DATA WHICH IS NOT BACKED UP!**

    boxbuilder_builder_instance_host=ubuntu

----

'boxbuilder\_build\_ami\_url' is the location from which the 'boxbuilder\_remote\_build\_ami' script will be downloaded
onto the EC2 builder instance.  Override it to use your custom 'boxbuilder\_build\_ami' script instead of
the default.

    boxbuilder_build_ami_url=http://github.com/thewoolleyman/boxbuilder/raw/master/boxbuilder_build_ami

----

'boxbuilder\_ec2\_api\_tools\_url' is the location from which the Amazon EC2 API Tools will be downloaded and extracted
to $HOME/.boxbuilder_ec2_tools/ec2-api-tools on your local filesystem .  Override it to use a custom location to download
the Amazon EC2 API Tools.

    boxbuilder_ec2_api_tools=http://s3.amazonaws.com/ec2-downloads/ec2-api-tools.zip

----

'boxbuilder\_terminate\_builder\_instance' is a boolean flag indicating whether a NON-GUARANTEED ATTEMPT should be
made to automatically terminated the EC2 builder instance when the script exits.  It is true by default.  Set it to
false if you want to rerun or debug boxbuilder on the build instance, or if you are using your own
'boxbuilder\_builder\_instance\_host'.

**WARNING: If you set 'boxbuilder\_builder\_instance\_host' to one of your own preexisting EC2 instances
and do NOT set 'boxbuilder\_terminate\_builder\_instance' to false, the instance 
WILL BE TERMINATED and you will LOSE ANY DATA WHICH IS NOT BACKED UP!**

    boxbuilder_terminate_builder_instance=true

----
&nbsp;


_EC2 Info_
==========

* Current Base AMI: ami-4b4ba522 - Ubuntu 10.04 LTS amd64 server (Lucid Lynx) (us-east-1)

* Alestic AMI list: http://alestic.com/
* Ubuntu AMI list: http://uec-images.ubuntu.com/releases/lucid/release/

----
&nbsp;


_Testing_
=========

Boxbuilder has a test suite which runs boxbuilder against a live machine, then asserts that it was
built correctly.  It requires the same SSH variables to be set as does the 'boxbuilder\_remote\_bootstrap'
('boxbuilder\_keypair', 'boxbuilder\_user', and 'boxbuilder\_host').  It will also read these and
any other variables set in ~/.boxbuilderrc.

----
