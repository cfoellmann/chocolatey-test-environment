# Chocolatey Testing Environment

A testing environment setup similar to the [package-verifier](https://github.com/chocolatey/package-verifier/wiki) for testing packages. Over time this will add more Windows platforms for testing.

When creating packages or testing other parts of Chocolatey, this environment provides a good base for an independent testing minus any dependencies you may already have installed. It also allows you to completely destroy an environment and then just tear it down without worry about messing up something on your own system.

When creating packages, please review https://github.com/chocolatey/choco/wiki/CreatePackages

## Requirements

You need a computer with:

* a 64-bit processor and OS
* Intel VT-x [enabled](http://www.howtogeek.com/213795/how-to-enable-intel-vt-x-in-your-computers-bios-or-uefi-firmware/) (usually not an issue if your computer is newer than 2011). This is necessary because we are using 64bit VMs.
* Hyper-V may need to be disabled for Virtualbox to work properly if your computer is a Windows box.
* At least 10GB of free space.

## Setup

To get started, ensure you have the following installed:
 * Vagrant 1.8.1+ - linked clones is the huge reason here. You can technically use any version of Vagrant 1.3.5+. But you will get the best performance with 1.8.x.
 * Virtualbox 4.3.28+ (5.x may have issues, so try to stay in 4.3.x series)
 * vagrant sahara plugin (`vagrant plugin install sahara`)

You can also install Vagrant/Virtualbox on Windows by running `choco install packages.config`

## Running Verification Manually

**NOTE**: The CDN for packages on https://chocolatey.org will only update every 30 minutes. This means if you just pushed an updated version, within 30 minutes from the last access time of the package it will be updated. This is why the validator and verifier wait for 31 minutes prior to testing a package.

### Preparing the Testing Environment

 * Ensure setup above is good on your machine.
 * Open a command line (`PowerShell.exe`/`cmd.exe` on Windows, `bash` everywhere else) and navigate to this directory of the repository. You know you are in the right place when you do a `dir` or `ls` and `Vagrantfile` is in your path.
   * No idea if bash on Windows (through Git/CygWin) is supported. If you run into issues, it is better to just use `PowerShell.exe` or `cmd.exe`. Please do not file issues stating it doesn't work.
 * Run `vagrant up` to prepare the machine for testing. Note that it will take quite awhile the first time you need to download the [box from Atlas](https://atlas.hashicorp.com/ferventcoder/boxes/win2012r2-x64-nocm). Once it has downloaded it will import the box and apply the scripts and configurations to the box as listed inside the `Vagrantfile`.
 * Now the box is ready for you to start testing against. Run `vagrant sandbox on`.

### Testing a Package

For testing a package, you have two avenues. For a locally built package, you can drop the package into the `vagrant/packages` folder - it is shared with the box as `C:\packages`, so you can run a command on the box or with the inline provisioner (recommended as it is a closer match to the verifier) using `--source c:\packages` as an argument for installation. If you are trying to reproduce/investigate a problem with a package already up on the website, you can use `--version number` with your install arguments and that will let you install a package that is not listed (in most cases not yet approved).

 * Uncomment the final string in the vagrant file and update it using one of the methods above, then run `vagrant provision`.
 * Watch the output and go to the box for further inspection if necessary.

### Make Changes and Retest

When you need to investigate making changes and rerunning the tests, remember that we took a snapshot of the vagrant machine (the virtual machine), so we can rollback to the earlier state each time and move forward with testing changes without the possibility of lingering artifacts. This is why we are using the sahara vagrant plugin, it allows us to take a snapshot and then revert the virtual machine back to the previous state.

When you are ready to reset to the state just before installing:

 * Run `vagrant sandbox rollback`.

### Tearing Down the Testing Environment
**NOTE**: At any time you can stop or remove the box with `vagrant suspend`, `vagrant halt` and/or `vagrant destroy`. For more information on vagrant commands, see the [Vagrant Docs](http://docs.vagrantup.com/v2/cli/index.html).

 * If you are finished with the vagrant box, you can remove your temporary copy with `vagrant destroy`.

## Differences Between This and Package Verifier Service

There are a couple of difference between the [verifier service]() and this environment.

 * The verifier is run without the GUI - meaning it is run in a headless state. There is no box to interact with.
 * The verifier only runs against Windows 2012 R2 currently. This repo is adding more boxes as they become available.
 * The verifier times out on waiting for a command after 12 minutes.
 * Synced folders are different - the verifier syncs the .chocolatey folder to gather the package information files.
 * Specific VM settings are different (for performance):
    * No GUI (as previously mentioned) - `v.gui = false`
    * 6GB RAM - `v.customize ["modifyvm", :id, "--memory", "6144"]`
    * 4 CPUs - `v.customize ["modifyvm", :id, "--cpus", "4"]`
    * Clipboard disabled - `v.customize ["modifyvm", :id, "--clipboard", "disabled"]`
    * Drag and Drop disabled - `v.customize ["modifyvm", :id, "--draganddrop", "disabled"]`

## Troubleshooting

You get this error: "A Vagrant environment or target machine is required to run this command. Run `vagrant init` to create a new Vagrant environment. Or, get an ID of a target machine from `vagrant global-status` to run this command on. A final option is to change to a directory with a Vagrantfile and to try again." - please ensure you are on the correct working directory (where this ReadMe and `Vagrantfile` is) of this repo and try again.
