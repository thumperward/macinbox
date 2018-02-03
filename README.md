# macinbox

Puts macOS in a Vagrant VMware Fusion box.

## System Requirements

* macOS 10.13 High Sierra host operating system
* At least 8 GB RAM (16 GB recommended)
* At least 2 cores (4 recommended)
* At least 30 GB of available disk space (60 GB recommended)

## Dependencies

The following software is required. Versions other than those mentioned may work, but these are the latest versions tested:

* VMware Fusion 10.1.1
* Vagrant 2.0.2
* Vagrant VMware Fusion Plugin 5.0.4
* macOS 10.13.3 High Sierra installer application

[Get VMware Fusion](http://www.vmware.com/products/fusion.html)
//
[Get Vagrant](https://www.vagrantup.com/)
//
[Get Vagrant VMware Fusion Plugin](https://www.vagrantup.com/vmware/)
//
[Get macOS 10.13 High Sierra installer application](http://appstore.com/mac/macoshighsierra)

## Installation

Install the gem:

    $ sudo gem install macinbox

## Basic Usage

Run with `sudo` and no arguments, the `macinbox` script will create and add a Vagrant VMware box named 'macinbox' using the installer app it finds in the default location:

    $ sudo macinbox

Please be patient, as this may take a while. (On a 2.5 GHz MacBookPro11,5 it takes about 11 minutes, 30 seconds.) After the script completes you can create a new Vagrant environment with the box and start using it:

    $ vagrant init macinbox && vagrant up

A few moments after running this command you will see your virtual machine's display appear in a fullscreen window. A couple of minutes later, after the virtual machine completes booting, you will see the desktop of the `vagrant` user and can begin using the virtual machine.

## Advanced Usage

To see the advanced options, pass the `--help` option:

```
$ macinbox --help

Usage: macinbox [options]
    -n, --name NAME                  Name of the box         (default: macinbox)
    -d, --disk SIZE                  Size (GB) of the disk   (default: 64)
    -m, --memory SIZE                Size (MB) of the memory (default: 2048)
    -c, --cpu COUNT                  Number of virtual cores (default: 2)
    -s, --short NAME                 Short name of the user  (default: vagrant)
    -f, --full NAME                  Full name of the user   (default: Vagrant)
    -p, --password PASSWORD          Password of the user    (default: vagrant)
        --installer PATH             Path to the macOS installer app
        --vmware PATH                Path to the VMware Fusion app
        --no-auto-login              Disable auto login
        --no-skip-mini-buddy         Show the mini buddy on first login
        --no-hidpi                   Disable HiDPI resolutions
        --no-fullscreen              Display the virtual machine GUI in a window
        --no-gui                     Disable the GUI
        --debug                      Enable debug mode
    -h, --help
```

Enabling debug mode causes the intermediate files (disk image, VMDK, and box) to be preserved after the script exits rather than being cleaned up. WARNING!!! These intermediate files are very large and you can run out of disk space very quickly when using this option.

Here is an advanced example which creates and adds a box named 'macinbox-large-nogui' with 4 cores, 8 GB or RAM, and a 128 GB disk; turns off auto login; and prevents the VMware GUI from being shown when the VM is started:

    $ macinbox -n macinbox-large-nogui -c 4 -m 8192 -d 128 --no-auto-login --no-gui

## Retina Display and HiDPI Support

By default macinbox will configure the guest OS to have HiDPI resolutions enabled, and configure the virtual machine to use the native display resolution.  You can disable this behavior using the --no-hidpi option.

## Details

This script performs the following actions:

1. Creates a new blank disk image
1. Installs macOS
1. Installs the VMware tools
1. Updates the SystemPolicyConfiguration KextPolicy to allow the VMware tools kernel extension to load automatically
1. Adds an .InstallerConfiguration file to automate the Setup Assistant app and create a user account on first boot
1. Enables password-less sudo
1. Enables sshd
1. Adds an rc.installer_cleanup script which waits for the user account to be created on first boot and then installs the default insecure Vagrant SSH key in the user's home directory
1. Enables HiDPI resolutions
1. Converts the image into a VMDK
1. Creates a Vagrant box for the VMware provider using the VMDK
1. Adds the box to Vagrant

This script is intended to do everything that needs to be done to a fresh install of macOS before the first boot to turn it into a Vagrant VMware box that boots macOS with a seamless user experience. However, this script is also intended to the do the least amount of configuration possible. Nothing is done that could instead be deferred to a provisioning step in a Vagrantfile or packer template.

## Acknowledgements

This project was inspired by the great work of others:

* http://grahamgilbert.com/blog/2013/08/23/creating-an-os-x-base-box-for-vagrant-with-packer/
* http://heavyindustries.io/blog/2015/07/05/create_osx_vagrant_vmware_box.html
* https://spin.atomicobject.com/2015/11/17/vagrant-osx/
* https://github.com/timsutton/osx-vm-templates
* https://github.com/boxcutter/macos
* https://github.com/chilcote/vfuse
* http://www.modtitan.com/2017/10/lazy-vm-building-hacks-with-autodmg-and.html

## Why?

My preferred operating system is macOS, and ever since I started using Vagrant, I thought that it would be nice to have be able to boot a macOS box as easily as a Linux box. However, it wasn't until I was watching an episode of Mr. Robot that I was finally inspired to figure out how to make it happen. In the episode, Elliot is shown quickly booting what appeared to be a virtual machine running a Linux desktop environment in order to examine the contents of an untrusted CD-ROM, and I thought, "I want to be able to do that kind of thing with macOS!".

In researching prior art, I discovered Timothy Sutton's `osx-vm-templates` project and realized that I would be able to use those scripts and packer templates to accomplish my goal. However, after using those the scripts and templates a few times and trying to customize them, I found that they didn't always work reliably for me. I began trying to understand how they worked so that I could make them more reliable and customizable, and created [vagrant-box-macos](https://github.com/bacongravy/vagrant-box-macos).

With the release of macOS 10.12.4, however, the prevailing techniques for customizing OS installs were hampered by a new requirement that all packages be signed by Apple. Since supporting macOS 10.13 High Sierra (and later) was going to require a major change to the scripts anyways, I decided to create 'macinbox' as a simpler, more streamlined approach to building macOS boxes. The previous scripts were more flexible, but 'macinbox' is faster and more reliable.

I chose to support only VMware Fusion boxes because of the Vagrant VMware plugin support for linked clones and shared folders, and because in my experience I have found that macOS virtualizes better in VMware Fusion than the alternatives.

## Development

After checking out the repo, run `bin/setup` to install dependencies. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To run `macinbox` directly from the root of the git workspace without installing the gem, run `sudo bundle exec macinbox`.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/bacongravy/macinbox.
