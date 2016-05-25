---
layout: post
title: How To Install Ubuntu in a Virtual Machine on Windows
---

This tutorial will teach you how to install Ubuntu Linux in a virtual machine on Windows. This is a good way to start setting up a programming development environment because many developer tools and programs are hard to install on Windows or only run on a Linux like Ubuntu. VirtualBox is a program that emulates a computer, so that you can install different operating systems onto the virtual computer. Ubuntu is a Linux distribution that is easy to use and configure which makes it ideal for a beginner to use. When you combine these two technologies together, you get an accessible way to develop programs on Windows without the hassle of downloading and installing Linux on your real non-virtualized computer.

# Table of Contents
* TOC
{:toc}

# Downloading VirtualBox

Go to [Downloads - Oracle VM VirtualBox] to download the virtualization program.

![VirtualBox Download Page]

Select the download link for Windows hosts because the computer that is "hosting" the emulation is a Windows machine.

# Installing VirtualBox

Just click `Next` and go through the installation with the default options.

![VirtualBox Installation First Step]

A `Warning: Network Interfaces` dialogue may appear, but don't worry -- just proceed with the installation. All this means is that VirtualBox is going to install drivers, so that the virtual computer can access the internet by connecting to the your real non-virtualized computer's internet.

![VirtualBox Network Interfaces Warning]

Another dialogue may pop up titled "Windows Security" asking whether you "Would like to install this device software?". Proceed with the installation because this is also part of the driver installation process.

# Downloading Ubuntu

Go to [Download Ubuntu Desktop] and click the `Download` button on the right to download the desktop version of Ubuntu (There are other versions to accommodate use cases like servers and phones).

![Ubuntu Download Page]

Unless you would like to donate, you can skip the donation

![Skip Donation]

# Running Virtualbox

Open VirtualBox and select the `New` button.

![VirtualBox Program]

Choose these values and title the virtual machine what you want.

![VirtualBox Naming]

Depending on how much ram your computer has, you want to select as much as possible for the virtual machine, but not more than half the available ram. I have 8GB of ram on my computer, so I don't have to worry that much about how much ram I select. A user with 2GB of ram might want to select 512MB in order to keep their physical computer running fairly smoothly while operating the virtual machine.

![VirtualBox Ram Size]

I would keep the default values for this, but maybe increase the hard drive size if possible. VirtualBox creates a virtual hard drive for the virtual machine to keep the files separated from each other. The virtual machine can't see your personal files with this setup.

![VirtualBox Hard Drive Size]

Keep the default option here and click `Next` -- these are just different virtual hard drive formats to choose from.

![VirtualBox Hard Drive Type]

I generally prefer the default option because it allows me to save space on my physical hard drive, but you can choose the other option if you'd like the virtual hard drive to stay a fixed size.

![VirtualBox Hard Drive Allocation]

You can change the location and size here if you want, but I like to keep it default.

![VirtualBox Hard Drive Location]

Congratulations! You now have a virtual machine set up for Ubuntu!

# Installing Ubuntu

Click Start to power on the machine.

![VirtualBox Main Window with Ubuntu]

This window should pop up. Click on the folder icon to select the Ubuntu .iso file you downloaded earlier.

![VirtualBox Startup Disk]

![Select Ubuntu Download]

When this screen comes up, it means that the virtual machine is booting successfully.

![VirtualBox Machine Starting]

This screen means that Ubuntu has booted up successfully.

If you want to just try Ubuntu, you can choose the try option. However, this guide will go over the installation of Ubuntu, so choose that option if you would like to continue and have Ubuntu persist when you reboot it.

![Ubuntu Started]

Select these options if you want. Downloading updates now means that it takes longer to install, but you don't have to update later. Third-party software isn't necessarily open source (meaning all of the programming code is available for you to view and edit yourself), and can be poorly made. But, it is usually required for many drivers that are essential for using the computer.

![Ubuntu First Step]

Even though this window says that the first option will erase the whole disk, it means the *virtual* disk. So you don't need to worry about wiping all of your personal files. Select the first option unless you'd like to use another method yourself.

Select Install `Now` to start the process.

![Ubuntu Disk Options]

Click `Continue`.

![Ubuntu Disk Verification]

Select your location so that Ubuntu can accurately set the system clock.

![Ubuntu Timezone]

Select your keyboard preference and language.

![Ubuntu Keyboard]

Set your account details to protect your virtual machine and give it a user account. Write this information down because you'll need it later to log in after it reboots.

![Ubuntu Login]

You should see this screen as Ubuntu installs and downloads files to your virtual machine's hard drive.

![Ubuntu Installing]

Click `Restart` to finish installing Ubuntu.

![Ubuntu Restart]

If this image comes up, you'll have to reset the virtual machine manually.

![Ubuntu Hang]

Click `Machine`, then `Reset`.

![VirtualBox Reset]

Confirm you want to reset.

![VirtualBox Reset Confirm]

Now your Ubuntu machine should be up and running!

![Ubuntu Installed]

If your screen isn't scaling right (see the scrollbars at the right and bottom), follow these steps. Otherwise, you're ready to go!

![VirtualBox Screen Scale]

Open up a terminal.

![Ubuntu Terminal]

Type in `sudo apt-get install virtualbox-guest-dkms` and hit `Enter`. Then type in the password you made for your account earlier and it `Enter`.

- `sudo` gives you root (administrator) privileges to install packages.
- `apt-get` installs programs from Ubuntu's repositories.
- `virtualbox-guest-dkms` is the name of the program that gives your Ubuntu virtual machine the right methods to change the display.

![Ubuntu VirtualBox Guest DKMS]

Press `y` and `Enter` to confirm installation.

![Ubuntu VirtualBox Guest DKMS Y]

After this has finished installing, this will show up with a blinking cursor (with the settings you used for your user account).

![Ubuntu DKMS Finished]

Reboot to finish the changes.

![Ubuntu Reboot]

![Ubuntu Reboot Confirm]

# Setting up the virtual machine to handle server capabilities

If you want to use your virtual machine as a server or to develop web applications and server programs, you have to switch the virtual machine's networking to bridged.

First, go to the settings in VirtualBox

![VirtualBox Settings]

Then, select the `Network` option on the side and select `Bridged Adapter` from the `Attached to:` dropdown in the center.

![VirtualBox Networking]

# Conclusion

Now you have a capable Linux virtual machine installed. You could learn how to program on it, run open source software, or learn how to do security testing. Those are just a few options of what you can do in this Ubuntu installation.

[Downloads - Oracle VM VirtualBox]: https://www.virtualbox.org/wiki/Downloads
[VirtualBox Download Page]: {{ site.url }}/images/virtualbox-download-page.jpg
[VirtualBox Installation First Step]: {{ site.url }}/images/virtualbox-installation-first-step.jpg
[VirtualBox Network Interfaces Warning]: {{ site.url }}/images/virtualbox-network-interfaces-warning.jpg
[Download Ubuntu Desktop]: http://www.ubuntu.com/download/desktop
[Ubuntu Download Page]: {{ site.url }}/images/download-ubuntu-page.jpg
[Skip Donation]: {{ site.url }}/images/skip-donation.jpg
[VirtualBox Program]: {{ site.url }}/images/after-virtualbox-install.jpg
[VirtualBox Naming]: {{ site.url }}/images/virtualbox-ubuntu-naming.jpg
[VirtualBox Ram Size]: {{ site.url }}/images/virtualbox-ram-size.jpg
[VirtualBox Hard Drive Size]: {{ site.url }}/images/virtualbox-hard-drive-size.jpg
[VirtualBox Hard Drive Type]: {{ site.url }}/images/virtualbox-hard-drive-type.jpg
[VirtualBox Hard Drive Allocation]: {{ site.url }}/images/virtualbox-hard-drive-allocation.JPG
[VirtualBox Hard Drive Location]: {{ site.url }}/images/virtualbox-hard-drive-location.jpg
[VirtualBox Main Window with Ubuntu]: {{ site.url }}/images/virtualbox-main-window-with-ubuntu.jpg
[VirtualBox Startup Disk]: {{ site.url }}/images/virtualbox-startup-disk.jpg
[Select Ubuntu Download]: {{ site.url }}/images/select-ubuntu-download.jpg
[VirtualBox Machine Starting]: {{ site.url }}/images/virtualbox-machine-starting.jpg
[Ubuntu Started]: {{ site.url }}/images/virtualbox-ubuntu-started.jpg
[Ubuntu First Step]: {{ site.url }}/images/ubuntu-first-step.png
[Ubuntu Disk Options]: {{ site.url }}/images/ubuntu-disk-options.png
[Ubuntu Disk Verification]: {{ site.url }}/images/ubuntu-disk-verification.png
[Ubuntu Timezone]: {{ site.url }}/images/ubuntu-timezone.png
[Ubuntu Keyboard]: {{ site.url }}/images/ubuntu-keyboard.png
[Ubuntu Login]: {{ site.url }}/images/ubuntu-login.png
[Ubuntu Installing]: {{ site.url }}/images/ubuntu-installing.png
[Ubuntu Restart]: {{ site.url }}/images/ubuntu-restart.png
[Ubuntu Installed]: {{ site.url }}/images/ubuntu-installed.png
[VirtualBox Screen Scale]: {{ site.url }}/images/virtualbox-ubuntu-screen-scale.jpg
[Ubuntu Terminal]: {{ site.url }}/images/ubuntu-terminal.png
[Ubuntu VirtualBox Guest DKMS]: {{ site.url }}/images/ubuntu-virtualbox-guest-dkms.png
[Ubuntu VirtualBox Guest DKMS Y]: {{ site.url }}/images/ubuntu-virtualbox-guest-dkms-y.png
[Ubuntu DKMS Finished]: {{ site.url }}/images/ubuntu-dkms-installed.jpg
[Ubuntu Reboot]: {{ site.url }}/images/ubuntu-reboot.png
[Ubuntu Reboot Confirm]: {{ site.url }}/images/ubuntu-reboot-confirm.png
[Ubuntu Hang]: {{ site.url }}/images/ubuntu-hang.png
[VirtualBox Reset]: {{ site.url }}/images/virtualbox-reset.jpg
[VirtualBox Reset Confirm]: {{ site.url }}/images/virtualbox-reset-confirm.jpg
[VirtualBox Settings]: {{ site.url }}/images/virtualbox-settings.jpg
[VirtualBox Networking]: {{ site.url }}/images/virtualbox-network-bridged.jpg
