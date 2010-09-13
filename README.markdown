BOXBUILDER
==========

Boxbuilder builds boxes!  Basic Bash scripts to bootstrap barebones-OS boxes with
[chef-solo](http://wiki.opscode.com/display/chef/Chef+Solo).  It can also create
EC2 AMIs.  Currently it supports Debian/Ubuntu, CentOS, and OSX/Homebrew.   Pull requests with support for
new distros and package managers are welcome.

* Homepage: You're reading it - [http://github.com/thewoolleyman/boxbuilder](http://github.com/thewoolleyman/boxbuilder)
* Tracker Project: [http://www.pivotaltracker.com/projects/101913](http://www.pivotaltracker.com/projects/101913)
* Bug Reports/Feature Requests: [http://github.com/thewoolleyman/boxbuilder/issues](http://github.com/thewoolleyman/boxbuilder/issues)

**WARNING! BOXBUILDER INCURS EC2 RESOURCE CHARGES!
The 'build\_ami' scripts will automatically create EC2 instances, EBS volumes and EBS snapshots.
Boxbuilder will ATTEMPT TO automatically terminate any EC2 instance it has automatically started,
but if the scripts fail or are killed the might not be removed.
It is YOUR RESPONSIBILITY to confirm that any EC2 resources which boxbuilder creates are
terminated even if boxbuilder fails to terminate them automatically.
If you do not YOU WILL BE CHARGED BY AMAZON FOR ANY RESOURCES UNTIL THEY ARE TERMINATED.
Learn how to delete any unused resources via the the EC2 AWS Management Console
before using the 'build\_ami' scripts: [https://console.aws.amazon.com/ec2/home](https://console.aws.amazon.com/ec2/home) .
Resources which boxbuilder automatically starts will be identified by the string
'boxbuilder\_temp\_builder\_instance\_safe\_to\_terminate' in the resource's
User Data instance attribute or Snapshot Description.
The User Data instance attribute is not visible via the EC2 AWS Management Console,
but can be seen using this command:
'ec2-describe-instance-attribute {instance id} --user-data'.**

----
&nbsp;


_Instructions_
==============

**Step 1. Run the script to create a default box or AMI**

By default, boxbuilder runs using a example default config file at
[http://github.com/thewoolleyman/boxbuilder/raw/master/boxbuilderrc\_download\_example](http://github.com/thewoolleyman/boxbuilder/raw/master/boxbuilderrc_download_example),
which in turn references example default chef repositories.

Your first step should be to run boxbuilder with the default config and ensure you can create a example test box or AMI.  This
will verify that your EC2 account and credentials are properly configured.

To build a box, download and run
'[boxbuilder\_bootstrap](http://github.com/thewoolleyman/boxbuilder/raw/master/boxbuilder_bootstrap)'
on a clean Debian/Ubuntu, CentOS, or OSX box, or download and run
'[boxbuilder\_remote\_bootstrap](http://github.com/thewoolleyman/boxbuilder/raw/master/boxbuilder_remote_bootstrap)'
from your local shell.

To build an AMI image (of a 'box' built by boxbuilder in a chroot jail on an EC2 box), download and run
'[boxbuilder\_build\_ami](http://github.com/thewoolleyman/boxbuilder/raw/master/boxbuilder_build_ami)' on a running AMI
instance, or download and run
'[boxbuilder\_remote_build\_ami](http://github.com/thewoolleyman/boxbuilder/raw/master/boxbuilder_remote_build_ami)'
from your local shell.

You will be prompted to enter all required variables for the script you are running, such as
your EC2 credentials, SSH keys, etc.  See details on supported and required variables for
each script in the sections below.

Check the output of the script for a success message, and/or any errors.  Log into the newly-built
box (or an instance started from your newly-built AMI), and verify everything worked.  To verify,
you can check that the example test chef recipes created touchfiles in the home directory:
[http://github.com/thewoolleyman/boxbuilder\_example\_main\_chef\_repo/blob/master/cookbooks/boxbuilder\_example\_main\_cookbook/recipes/touchfile.rb](http://github.com/thewoolleyman/boxbuilder_example_main_chef_repo/blob/master/cookbooks/boxbuilder_example_main_cookbook/recipes/touchfile.rb) and
[http://github.com/thewoolleyman/boxbuilder\_example\_dependency\_chef\_repo/blob/master/cookbooks/boxbuilder\_example\_dependency\_cookbook/recipes/touchfile.rb](http://github.com/thewoolleyman/boxbuilder_example_main_chef_repo/blob/master/cookbooks/boxbuilder_example_main_cookbook/recipes/touchfile.rb)

If you have problems, try setting 'boxbuilder\_debug' to true.

**Step 2. Create a custom box by using a custom config file and chef repos**

To build a custom box, you must override the variables found in the example default config file.
The easiest way to do this is to:

1. Copy the example boxbuilderrc\_download\_example config file to boxbuilderrc\_download and publish it in your own git repo (you can create one for free on github)
2. (optional) Edit the variables in it to point to your own chef repositories, config path, and json path (and any other variables you want to add/override).
3. (optional) Create or reuse custom chef cookbooks in your chef repositories at which you pointed.  If you wish, you can fork or copy the default ones and customize them.
4. Set/export the 'boxbuilderrc\_download\_url' variable to point to your custom boxbuilderrc\_download you published in your own git repo.  You can set/export boxbuilderrc\_download\_url in the following ways:
    * Directly on the command line (an error will be raised if an existing .boxbuilderrc exists and contains a different boxbuilderrc\_download\_url)
    * In $HOME/.boxbuilderrc if you are running 'boxbuilder\_bootstrap' or 'boxbuilder'
    * In $HOME/.boxbuilder\_remote\_bootstraprc if you are running 'boxbuilder\_remote\_bootstrap'
    * In $HOME/.boxbuilder\_build\_amirc if you are running 'boxbuilder\_build\_ami'
    * In $HOME/.boxbuilder\_remote\_build\_amirc if you are running 'boxbuilder\_remote\_build\_ami'

Once you have your custom config file and (optionally) chef repos set up, run the script again and verify your chef cookbooks
and recipes build your custom box/AMI as expected.

If you have problems, try setting 'boxbuilder\_debug' to true.

**Step 3. Test/Debug/Improve your custom chef repos**

If you have built a box (not an AMI), you can log in to it and directly modify the cloned chef repos
under $HOME/.chef, and re-run $HOME/.boxbuilder/boxbuilder, or run chef-solo directly with the
proper options to specify your chef json and config files.  If you
add a writeable origin to your git repos under $HOME/.chef, you can check your changes in directly
from the box.

AMI builds are more complex to create and test.  Most importantly, you must be aware that the AMI build
runs in a chroot jail.  This means that some chef actions will not work as expected or at all.  To
work around this, you can have a "chroot" recipe which runs after all others, and prevents any actions
which are not chroot-safe from running.  For an example of this, see the Rails CI build (warning: incomplete and may move):
[http://github.com/thewoolleyman/railsci\_chef\_repo/blob/master/cookbooks/railsci/chroot/recipes/default.rb](http://github.com/thewoolleyman/railsci_chef_repo/blob/master/cookbooks/railsci/chroot/recipes/default.rb)

The recommended way to work directly on a chef repo which builds an AMI is to run 'boxbuilder\_remote\_build\_ami'
with the 'boxbuilder\_terminate\_ec2\_resources' variable set to 'false'.  This will leave the EC2 builder instance
running, and you can then log in and run $HOME/.boxbuilder/boxbuilder\_build\_ami repeatedly.  **WARNING: THIS
MEANS YOU MUST TERMINATE ALL CREATED EC2 INSTANCES MANUALLY OR YOU WILL CONTINUE TO BE CHARGED FOR THEM!**
Also, be aware there are sometimes intermittent errors related to reusing the
same chroot resources for multiple runs of 'boxbuilder\_build\_ami'.

If you suspect a problem with boxbuilder itself, you can set 'boxbuilder\_debug' to true, and you will get
verbose logging of script execution.  This debug flag is propagated to all other boxbuilder scripts
which are sourced locally or invoked remotely.

----
&nbsp;


_Overview_
==========

Boxbuilder consists of simple layered bash scripts which invoke each other locally or remotely, starting
EC2 instances if required.  Here's the invocation order of the scripts for the two main tasks
(building a box or building an AMI).  You can, of course, run and re-run any of these scripts directly
by logging into the box being built or the EC2 image building the AMI.

Building a Box:

1. 'boxbuilder\_remote\_bootstrap' (invoked from any Bash shell) issues SSH commands on the box being built to download and invoke...
2. 'boxbuilder\_bootstrap' (on the box being built), which clones a boxbuilder git repo, and invokes...
3. 'boxbuilder' (on the box being built) from the cloned repo, which builds the box according to the config specified in boxbuilderrc\_download\_url.

Building an AMI Image: 

1. 'boxbuilder\_remote\_build\_ami' (invoked from any Bash shell with Java) installs the EC2 tools, starts an EC2 instance, and issues SSH commands on it built to download and invoke...
2. 'boxbuilder\_build\_ami' (on the EC2 instance), which creates a chroot jail, and runs commands in the chroot jail to download and invoke...
3. 'boxbuilder\_bootstrap' (in the chroot jail), which clones a boxbuilder git repo, and invokes...
4. 'boxbuilder' (in the chroot jail) to build a 'box' in the chroot jail according to the config specified in boxbuilderrc\_download\_url, which then exits and returns control to...
5. 'boxbuilder\_bootstrap' (in the chroot jail), which exits and returns control to...
6. 'boxbuilder\_build\_ami' (on the EC2 instance), which continues to create an AMI image of the chroot jail.



----
&nbsp;


_Configuration: Flexible, Automatically Downloaded, Auto-Updating, Easily Overridable, With Example Defaults_
======================================================================

The configuration approach for boxbuilder is somewhat unorthodox, but it is designed to meet the following goals:

* Flexible: Several different ways to specify configuration - in a manually edited file, in an auto-downloaded file
  which lives in source control, in a file on a shared drive, etc...
* Automatically Downloaded: If you specify 'boxbuilderrc\_download\_url' with a URL to your 'boxbuilderrc_download' config file (which can be in a git repo),
  it will be automatically downloaded and used.
* Auto-Updating: The first time boxbuilder is run on a box, it creates a config file which will automatically re-download
  your config file from 'boxbuilderrc\_download\_url' every time boxbuilder is run.
* Easily Overridable: There are several different points at which you can set or override boxbuilder config environment variables,
  in the config files or on the command line.
* Example Defaults: When run without specifying a custom config file url in 'boxbuilderrc\_download\_url', boxbuilder will default to
  using a simple example configuration file which points to example chef repositories.  This makes it easy to see how
  boxbuilder works before you start creating custom configs and chef repos to build your own custom boxes.

Here's the main features and supported usage for the various boxbuilder config approaches:

* The boxbuilder script itself and the other scripts which invoke it locally or
  remotely are controlled via configuration values set in environment variables.
  Some have overridable defaults.  Others are required, and you will be prompted for
  them if they are not set.
* Environment variables can also be specified in the config file $HOME/.boxbuilderrc, or
  files which it sources (executes).
* If it does not already exist, $HOME/.boxbuilderrc will automatically be created
  with commands to automatically set, download and source (execute)
  $HOME/.boxbuilderrc\_download from the url specified in the 'boxbuilderrc\_download\_url'
  environment variable.  By default, it points to
  the '[boxbuilderrc\_download\_example](http://github.com/thewoolleyman/boxbuilder/raw/master/boxbuilderrc_download_example)'
  file at [http://github.com/thewoolleyman/boxbuilder/raw/master/boxbuilderrc\_download\_example](http://github.com/thewoolleyman/boxbuilder/raw/master/boxbuilderrc_download_example)
* The auto-creation of a $HOME/.boxbuilderrc file is a simple "auto-update" mechanism, which
  will result in the latest version of the config at 'boxbuilderrc\_download\_url' to be automatically
  downloaded and used every time boxbuilder is run.
* This simple config-file-based approach allows you to have standard config files for different
  machines stored in source control and ensure the latest version is always
  'auto-updated' to $HOME/.boxbuilderrc\_download.  This
  means your config is always in source control.
* Or, if you wish, you can add local overrides to the bottom of $HOME/.boxbuilderrc after it sources
  $HOME/.boxbuilderrc\_download.  This is useful if you want to test or debug by re-running the 'boxbuilder'
  script locally on the box being built.
* If you prefer not to use the 'auto-update' functionality, you can are free to edit the
  $HOME/.boxbuilderrc file to not automatically download and source $HOME/.boxbuilderrc\_download.
  With this approach, you can set all variables directly in $HOME/.boxbuilderrc, 
  set them via 'boxbuilder\_config' (see next bullet), or in some other way you choose.
* Environment variables can also be set or overridden via the 'boxbuilder\_config'
  variable.  The contents of this variable will be evaluated as a line of bash scripting
  AFTER the $HOME/.boxbuilderrc config file is sourced.
  This provides another easy command-line-based approach to override environment variables
  when you are using the 'remote' scripts to set up a remote box or the
  'boxbuilder' or 'boxbuilder\_bootstrap' scripts directly.
* Only the 'boxbuilder' (and 'boxbuilder_bootstrap') scripts directly source (execute)
  the $HOME/.boxbuilderrc file, because they are the only scripts which execute on the
  box (or AMI chroot jail image) being built.  All the other scripts (boxbuilder\_remote\_bootstrap,
  boxbuilder\_build\_ami, and boxbuilder\_remote\_build\_ami) read their own dedicated
  config files, named respectively .boxbuilder\_remote\_bootstraprc,
  .boxbuilder\_build\_amirc, and .boxbuilder\_remote\_build\_amirc.

Here's an ordered summary of the process for reading configuration from various sources
when the 'boxbuilder' script runs.  You can add/override environment variables at any point:

1. Any pre-existing variables are, naturally, already set before 'boxbuilder' is invoked
2. If $HOME/.boxbuilderrc doesn't exist, it is created by default, with commands to automatically download and source $HOME/.boxbuilderrc\_download
   from 'boxbuilderrc_download_url'.  Then it is sourced (executed).  If $HOME/.boxbuilderrc DOES exist, it is never touched or modified.
   You can always delete it and allow it to be recreated, or manually edit it.
3. You can edit and re-publish the config file at the auto-updated 'boxbuilderrc\_download\_url' specified in $HOME/.boxbuilderrc
4. You can add any manual, local overrides to the bottom of $HOME/.boxbuilderrc
5. $HOME/.boxbuilderrc is sourced by the 'boxbuilder' script
6. The contents of 'boxbuilder\_config' are evaluated.
7. Any environment variables not set by this point will either have a default value set, or will
   cause the script(s) to exit with a prompt if they are required.

----
&nbsp;


_boxbuilder\_bootstrap script_
==============================

'boxbuilder\_bootstrap' is a single downloadable helper script which will check out the
boxbuilder project to $HOME/.boxbuilder, and run the main '$HOME/.boxbuilder/boxbuilder' script.  It
is intended to be easily invoked on a clean box via wget or curl with a bash one-liner.

For example, log in or SSH to the box being built, and paste the following:

    wget -O /tmp/boxbuilder_bootstrap http://github.com/thewoolleyman/boxbuilder/raw/master/boxbuilder_bootstrap && chmod +x /tmp/boxbuilder_bootstrap && /tmp/boxbuilder_bootstrap
    # Be sure to log out or source $HOME/.bashrc after the first build

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
$HOME/.boxbuilderrc.  They will be evaluated by the 'boxbuilder\_bootstrap' script, which
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
which you can export from your shell, or set in
$HOME/.boxbuilder\_remote\_bootstraprc:

    wget -O /tmp/boxbuilder_remote_bootstrap http://github.com/thewoolleyman/boxbuilder/raw/master/boxbuilder_remote_bootstrap && chmod +x /tmp/boxbuilder_remote_bootstrap && /tmp/boxbuilder_remote_bootstrap

Note that the config file is $HOME/.boxbuilder\_remote\_bootstraprc, NOT
$HOME/.boxbuilderrc (which is the config for the current box, not the
remote box being built).

----

boxbuilder\_remote\_bootstrap environment variables
---------------------------------------------------

**(REQUIRED to log into the remote box being built)** Set 'boxbuilder\_keypair' to
the path of your private key which will allow you to log in to the
box being built (you should already have your public key in $HOME/.ssh/authorized_keys).  Set 
'boxbuilder\_user' to the user, and 'boxbuilder\_host' to the hostname or IP address of the
box being built.

    boxbuilder_keypair=path_to_private_key
    boxbuilder_user=user_for_box_to_build
    boxbuilder_host=hostname_or_ip_of_box_to_build

----

The contents of the 'boxbuilder\_config' variable should be Bash commands to export and override
boxbuilder config variables from their default values or values which will be loaded from
$HOME/.boxbuilderrc on the box build built.  It will be evaluated by the 'boxbuilder\_remote\_bootstrap'
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

* Process all configuration variables (see configuration section above)
* Install basic packages required for next steps (which you can override/augment with chef)
* Install Ruby via [RVM](http://rvm.beginrescueend.com/)
* Install [chef-solo](http://wiki.opscode.com/display/chef/Chef+Solo)
* Download Chef repositories you specify which contain custom cookbooks, config, and JSON attributes.
* Create and run a script which executes chef-solo with the config and JSON attributes you specify

----

boxbuilder environment variables
--------------------------------

'boxbuilderrc\_download\_url' is the URL to a boxbuilder config script.  If $HOME/.boxbuilderrc does not exist,
it will be created with commands to automatically download 'boxbuilderrc\_download\_url' to
$HOME/.boxbuilderrc\_download on the box which is being built and source (execute) it.

To build a custom box, you can override 'boxbuilderrc\_download\_url' to point to a custom configuration file
which exports variables for your custom chef repo locations and config.

See more details in the instruction and configuration sections above.

    boxbuilderrc_download_url=http://github.com/thewoolleyman/boxbuilder/raw/master/boxbuilderrc_download_example

----

'boxbuilder\_prerequisite\_packages' is a space-separated list of packages which will be installed
on the box being built.  By default, it is the minimal set of packages required for RVM and Ruby
on a clean Debian/Ubuntu, CentOS, or OSX Homebrew install.  Other packages which will be used to other applications on the box
should be installed via chef, as part of the application setup cookbooks.  On Debian, you can
specify an exact version with {packagename}={packageversion}.  See the prerequisite_packages_{platform}()
functions in the boxbuilder script for the latest default packages.

    boxbuilder_prerequisite_packages={See the prerequisite_packages_<platform>() function in the boxbuilder script for the latest default packages}

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

    boxbuilder_chef_dir=$HOME/.chef

----

**(REQUIRED to specify your chef repos)** 'boxbuilder\_chef\_repos' is a space-delimited list of Chef Git repositories which will be automatically
downloaded by boxbuilder.  They will be checked out under $boxbuilder\_chef\_dir ($HOME/.chef) on the box which is being built.

You must override this variable to point to custom chef repo(s) in order for boxbuilder to build a custom box.

    boxbuilder_chef_repos=git://github.com/thewoolleyman/boxbuilder_example_main_chef_repo.git[ git://github.com/thewoolleyman/boxbuilder_example_dependency_chef_repo.git[ ...]]

----

**(REQUIRED to specify your chef config)** 'boxbuilder\_chef\_config\_path' is the path of the Chef config file. boxbuilder will use
this as the '--config' parameter when it automatically creates and runs a chef-solo script on the
box which is being built.  This should be a path to a file
in one of your 'boxbuilder\_chef\_repos'.

You must override this variable to point to a custom chef config path in one of the repos specified in 'boxbuilder\_chef\_repos'.

    boxbuilder_chef_config_path=$boxbuilder_chef_dir/boxbuilder_chef_repo/config/solo.rb

----

**(REQUIRED to specify your chef config)** 'boxbuilder\_chef\_json\_path' is the path of the Chef JSON attributes file. boxbuilder will use
this as the '--json-attributes' parameter when it automatically creates and runs a chef-solo script
on the box which is being built.  This should be a path to a file
in one of your 'boxbuilder\_chef\_repos'.

You must override this variable to point to a custom chef json path in one of the repos specified in 'boxbuilder\_chef\_repos'.

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

Environment variables can be set in $HOME/.boxbuilder\_build\_amirc (not
$HOME/.boxbuilderrc, that is the config for the current box, not the AMI
being built in the chroot jail).

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

**(NOT REQUIRED but you probably want to set it)** 'boxbuilder\_ami\_prefix' is a string which will be prepended to the name of the AMI
being created.  This should be some string which describes the purpose of the AMI.  It will be prepended to the following
info to create the complete AMI name: "-ubuntu-{release}-{codename}-{tag}-{arch}-{timestamp}"

    boxbuilder_ami_prefix="built-by-boxbuilder"

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
on a remote box without logging in to it.
You will be prompted to set required variables
which you can export from your shell, or set in
$HOME/.boxbuilder\_remote\_build\_amirc:

* Download the Amazon EC2 API tools to your local filesystem
* Automatically start an EC2 instance (called the 'builder instance') using your EC2 account and credentials (which you are required to obtain and specify)
* Upload your credentials to the builder instance
* Issue remote SSH commands to download and run 'boxbuilder\_remote\_build\_ami' on the builder instance (which then builds your AMI, see documentation for 'boxbuilder\_build\_ami')

Note that the config file is $HOME/.boxbuilder\_remote\_build\_amirc, NOT
$HOME/.boxbuilderrc (which is the config for the current box, not the
remote AMI being built).

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
current 32-bit and 64-bit AMI IDs, as well as the CentOS AMIs.  The default value is 'ami-4b4ba522' (64-bit Ubuntu).

    boxbuilder_builder_instance_ami_id=ami-4b4ba522

----

**(REQUIRED to match default ssh-able user for builder instance)** 'boxbuilder\_user' is the username which is
used to log into the EC2 builder instance and perform the remote AMI build via SSH.
It must match a user which is ssh-able by default on the builder instance which is started from the AMI specified in 'boxbuilder\_builder\_instance\_ami\_id'.

Set it to 'ubuntu' if you are using the standard Ubuntu EC2 AMIs to
start a builder instance (more distros will be supported in the future).

    boxbuilder_user={required to be exported directly or in .boxbuilderrc}

----

'boxbuilder\_builder\_instance\_id' is the instance id of the EC2 builder instance.  If it is not set,
an instance will automatically be created for you.  Boxbuilder will ATTEMPT to terminate the instance after
the AMI is built, but this is not guaranteed.

    boxbuilder_builder_instance_id="i-abcdef12"

----

'boxbuilder\_build\_ami\_url' is the location from which the 'boxbuilder\_remote\_build\_ami' script will be downloaded
onto the EC2 builder instance.  Override it to use your custom 'boxbuilder\_build\_ami' script instead of
the default.

    boxbuilder_build_ami_url=http://github.com/thewoolleyman/boxbuilder/raw/master/boxbuilder_build_ami

----

'boxbuilder\_ec2\_api\_tools\_url' is the location from which the Amazon EC2 API Tools will be downloaded and extracted
to $HOME/.boxbuilder_ec2_tools/ec2-api-tools on your local filesystem.  Override it to use a custom location to download
the Amazon EC2 API Tools.

    boxbuilder_ec2_api_tools=http://s3.amazonaws.com/ec2-downloads/ec2-api-tools.zip

----

'boxbuilder\_terminate\_ec2\_resources' is a boolean flag indicating whether a
NON-GUARANTEED ATTEMPT should be made to automatically terminate ALL EC2 builder instances
which boxbuilder automatically started.
Resources which boxbuilder automatically started will be identified by the string
'boxbuilder\_temp\_builder\_instance\_safe\_to\_terminate' in the resource's
User Data instance attribute or Snapshot Description.
The User Data instance attribute is not visible via the EC2 AWS Management Console,
but can be seen using this command:
'ec2-describe-instance-attribute {instance id} --user-data'.

It is true by default.  This is an attempt to prevent old builder instances from persisting after
boxbuilder runs and incurring charges on your EC2 account (see the warning at the top of this README).

Set it to false if you want leave the build instance running in order to rerun or debug boxbuilder
(you must set 'boxbuilder\_builder\_instance\_id' to prevent a build instance from automatically starting).

    boxbuilder_terminate_ec2_resources=true

----
&nbsp;


_EC2 Info_
==========

* Current Ubuntu Base AMI (default): ami-4b4ba522 - Ubuntu 10.04 LTS amd64 server (Lucid Lynx) (us-east-1)

* Alestic AMI list: [http://alestic.com](http://alestic.com)
* Ubuntu AMI list: [http://uec-images.ubuntu.com/releases/lucid/release](http://uec-images.ubuntu.com/releases/lucid/release)

----

* Current CentOS Base AMI: ami-4d42a924 - RightImage\_CentOS\_5.4\_x64\_v5.5.9\_EBS

* RightScale AMI list: [http://support.rightscale.com/18-Release_Notes/ServerTemplates\_and\_RightImages/Current#V5\_Images](http://support.rightscale.com/18-Release_Notes/ServerTemplates_and_RightImages/Current#V5_Images)
* Rightscale Image Setup Scripts: http://support.rightscale.com/index.php?title=06-FAQs/FAQ_0103_-_How_do_I_make_any_Amazon_Machine_Image_(AMI)_capable_of_running_RightScripts%3F
* CentOS OS Images: http://mirrors.usc.edu/pub/linux/distributions/centos/5.5/os/x86_64/images/
----
&nbsp;


_Known Issues_
==========

* Boxbuilder was developed and tested on OSX and Ubuntu.  It is intended to be portable across all Bash platforms,
  but there may be differences in Bash implementations.  Please report bugs!

* Boxbuilder is opinionated and focused.  It assumes you are going to install Ruby
  and Chef via RVM as a non-root user with no-password sudo priviliges
  via membership in the 'admin' group.  It gives you as much control as
  possible over these tasks, and leaves everything else up to your Chef
  recipes.  It is written in Bash, and tries to be as portable as
  possible across platforms and distros.  If there's something you
  think boxbuilder should do but doesn't, please speak up!

----
&nbsp;


_Testing_
=========

Continuous Integration for Boxbuilder is at [http://ci.pivotallabs.com:3333/builds/boxbuilder](http://ci.pivotallabs.com:3333/builds/boxbuilder)

Boxbuilder has an integration test in test/boxbuilder_test which does the following:

* Runs boxbuilder\_remote\_build\_ami to create an AMI with the default
  example configuration and chef repos
* Starts a new EC2 instance from the newly-built AMI
* Verifies the AMI was built correctly by performing assertions on the instance via SSH
* Terminates/deletes all test resources (instance, AMI, and snapshot)

test/boxbuilder\_test requires the same variables to be set as does the 'boxbuilder\_remote\_build\_ami'
script.  It will prompt with an error if any required variable is unset.  It will also read
any other variables set in $HOME/.boxbuilder\_testrc.

The test takes several minutes to run, because that is how long it takes boxbuilder\_remote\_build\_ami
to run.  Be patient.  The terminal will also be locked and prevent input while boxbuilder\_remote\_build\_ami
is running.

If you are hacking boxbuilder, and the test fails, try setting boxbuilder\_debug to true.  This
will give a very verbose output to help you find the failure.

----
&nbsp;


_Credits_
=========

AMI-building code is based on Eric Hammond's tutorial at http://alestic.com/2010/01/ec2-ebs-boot-ubuntu

Props to the folks on the ec2ubuntu mailing list and the bash irc
channel/wiki; they helped me a lot.

----
&nbsp;


_Who to Blame_
==============

Boxbuilder was designed and written by Chad Woolley: [thewoolleyman@gmail.com](mailto:thewoolleyman@gmail.com)


----
