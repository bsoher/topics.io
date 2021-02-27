---
sort: 5
---

# The Vespa Appliance 

## Creating a Vespa Virtual Appliance

We provide for our users a "Vespa appliance" which is a virtual machine instance preloaded with Vespa. The appliance can (we hope!) be loaded under any popular virtualization software like VirtualBox, VMWare, or Parallels.

This document explains how to recreate the instance should you need to.

This document doesn't go into as much detail as it could. That's because it will probably only be used every few years when the guest operating system in the appliance gets very out of date. Once that much time has elapsed, both VirtualBox and the guest operating system will probably have changed quite a bit, so it doesn't make sense for me to go into great detail about _exactly_
when and where to click, what you'll see, etc.

### Maintenance

Maintaining the appliance after you've created it is covered in the next section below.

### Warning

At some point while you're building the appliance, there's a good chance you will log in to one of the Vespa wikis. *Do not check the browser's "remember my password" option*. Remember that everything in this appliance will become public, including your password if you leave it in the appliance. Treat it like a terminal in an Internet cafe in a foreign country.

### Prerequisites and Assumptions

I built the appliance using [VirtualBox](https://www.virtualbox.org/). The operating system I installed was [Lubuntu 12.04](http://lubuntu.net/). This assumes that you have VirtualBox already installed and an ISO file for the guest operating system already downloaded. It also assumes you know how to use VirtualBox and how to do some basic things in Linux (like add software). 

Lubuntu is lightweight Ubuntu which is perfect for running in a virutal machine where the virtualized hardware is almost always subpar. Lubuntu uses less disk space, too, which reduces the size of the appliance that the user must download.

### Creating the Instance

Start VirtualBox and create a new instance. I accepted all default settings except that in the Virtual Disk Creation Wizard, I chose a disk type of VMDK. I'm not sure that was important. For size, I chose 40Gb, dynamically allocated. 

Install Lubuntu. I chose all the defaults until the "Who Are You" screen which requires input. Here's what I used -- 

 * Your name: vespa
 * Computer's (network) name: vespa-vm
 * Username: vespa
 * Password: vespavespa
 * Clicked "log in automatically" option

The install will copy a bunch of files and then reboot. After rebooting, the update manager will start. Allow it to gather update info and then download & install all available updates.

*Do NOT install your virtualization software's enhancements to the guest OS*. They won't work under other virtualization software. In VirtualBox these enhancements are called "guest additions". 

### Configure the Guest

*Change the desktop background to something bland if necessary.* I found the default desktop sufficiently bland.

*Add a desktop shortcut to Vespa's Web site.* Find the web browser in the 'start' menu and right click on it. I got a popup that said "Add to Desktop".

Under Linux, Desktop app launchers are just INI-style text files with a `.desktop` extension that follow the [Desktop Entry format](http://standards.freedesktop.org/desktop-entry-spec/desktop-entry-spec-latest.html). Right click on the icon to edit with Leafpad or whatever text editor is available. The INI keys you want to edit  (under the heading `[Desktop Entry]`) are `Name` and `Exec`. The former is the name of the icon (e.g. "Vespa Web site"). The latter is the command that will launch the app. It should already be something like `/usr/bin/chromium-browser`. Add the URL as a parameter to that: 

```
/usr/bin/chromium-browser https://github.com/vespa-mrs/vespa/
```

*Add a desktop shortcut to Vespa's mailing list.* Create another browser shortcut on the Desktop and then follow the instructions above. Note that when editing the INI file, one can add `\n` (those two literal characters -- backslash and 'n') in the name to force word wrap if the name doesn't wrap nicely. Use this as the URL: http://tech.groups.yahoo.com/group/vespa-mrs/


*Install Vespa and its dependencies*. When you finish installing Vespa, don't forget to create the desktop icons. (Doing so is a manual step under non-Windows operating systems.) 

*Download the [Analysis sample data](http://scion.duhs.duke.edu/vespa/analysis/raw-attachment/wiki/Tutorials/analysis_tutorial_data.zip).* Unpack it and put it on the Desktop under a name like, "Analysis Sample Data".

*Add ReadMe.txt to the desktop.* A suggested version is shown below.

*Arrange icons nicely on the desktop.* I found that if I selected a group of icons with the left mouse and then right clicked on the group, I got a popup menu that included "snap to grid". 

When I was done, ![screenshot.png my desktop looked like this](images/screeshot.png).


### Create the Appliance File
Shut down the instance. On the VirtualBox menu, select "Export Appliance" and follow the prompts. I was [given the option to populate some metadata](images/ExportScreeshot.png), which I did half-heartedly since I wasn't sure it was meaningful.



## Maintaining the Vespa Appliance

This document describes how to maintain the Vespa virtual appliance.

### How to Update

1. Boot the appliance. 
1. Run the operating system's software update.
1. Update Vespa, if necessary.
1. Create the appliance file - see above
1. Upload it to your home directory on scion.
1. SSH to scion, sudo to root and move the file --

    ```
    sudo mv /home/flip/VespaAppliance.ova /var/www/appliance
    ```
    
1. The file is owned by you and isn't readable by anyone else, so it can't be downloaded. Give everyone permission to read it --

    ```
    chmod +r /var/www/appliance/VespaAppliance.ova 
    ```
    
1. Update the Appliance History history below.

### Age

When you first publish a version of the appliance, it's up to date. There are three things that will cause it to get out of date --
1. Minor operating system changes (security fixes, etc.)
1. Major operating system changes (new OS versions)
1. Vespa changes (new versions)

The first minor operating system changes will probably come within 24 - 72 hours of publishing the appliance. In other words, keeping the appliance completely current would require publishing a new version every few days. This isn't realistic, so we will have to live with the fact that the appliance will not always be current.

The good news is that the appliance is never broken as a result of being out of date. It might provide a poor experience for the user, however. It might use an old version of Vespa, and therefore not be representative of what Vespa can do. Or the operating system might want to install hundreds of updates when all the user wants to do is play with Vespa.

That said, *when should one update the appliance and publish a new version?* The answer is: it depends. It depends on how out-of-date the appliance is, and that depends on the rate of change of the items in the list above, and that's neither constant nor entirely predictable.

Personally, I'd consider updating the appliance every month or so.


## Appliance README file

Vespa is a free software suite for magnetic resonance (MR) spectral simulation, MR RF pulse design, and spectral processing and analysis of MR data.
	
This virtual machine allows you to try Vespa without installing it yourself. Keep in mind that Vespa will run more slowly under this virtual machine than when it is installed directly on your computer.

Your username on this virutal machine is "vespa" and the password is "vespavespa".

For more information about Vespa, double click on the "Vespa Web Site" icon on the desktop, or visit https://github.com/vespa-mrs/vespa.io/


## Vespa Appliance History

This section is an informal history of the Vespa virtual appliance.

We keep one and only one version of the appliance at any given time. If we update it, the new version replaces the old one. The history below details when we released a new version, and what changed.

### The History 

 * 20 April 2016    - Third release.  Using Lubuntu 12.04 and Vespa 0.8.6.
 * 11 October 2012  - Second release. Using Lubuntu 12.04 and Vespa 0.6.0.
 * 6 September 2012 - First release!  Using Lubuntu 12.04 and Vespa 0.5.3.