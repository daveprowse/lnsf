# Part III: Updates & Upgrades

One of the most important ways to keep a system secure is to update it. Linux distributions such as Debian, Ubuntu, Red Hat, and most others continually release security updates. These are essential to the operating system's well-being. But not only that, updates to individual programs are important as well. For instance, programs that use SSH and SSL, or work as web servers, and so on. These should be updated ASAP, as long as updates are performed within your organization's guidelines (policies and procedures). 

Different Linux distributions use different package managers, and so, updating will vary from one distro to the next. Here's a quick breakdown of some of the command line-based package managers used by Linux systems:

- Debian Linux (and derivatives): **apt**
- Red Hat (and derivatives): **dnf**
- SuSE (and derivatives): **zypper**
- Arch (and derivatives): **pacman**

We'll focus on the Advanced Package Tool (APT) that is used by Debian and Ubuntu (and many others). 

The apt tool can be used to: check for updates, install individual updates, install all updates, upgrade to a new version of the OS, install individual programs, and more. It is a powerful tool and should be used accordingly - with caution.

### Lab 3-1
**Using apt**

In this lab we will show: how to update all programs at once; update individual programs; install individual programs; and cleanup the system when done. 

- Update all programs

	!!! warning
		The following procedure will update the entire system to the latest point release and will update any existing programs that have updates available. Use with caution!

	At a Debian client system type the following command:

	```
	apt update && apt upgrade -y
	```
	
	> Note:	You will need root or sudo access to accomplish this.

	This command does the following:

	1. Checks for updates based on the "sources.list" file.
	2. Downloads and installs all updates

	It does this automatically without any other user intervention. While this is okay for a client system (usually), we might need to use more caution when working with servers. 

- Update a server

	Here we will update a Debian server, but we will do so in a more step-by-step fashion. 

	First, let's run `apt update` by itself so that the system will search for updates.

	Next, lets view what those updates are:

	`apt list --upgradeable`

	**Example on Debian Server**
		
    Here's an example of the two previous commands:

		```
		root@deb51:~# apt update
		Hit:1 http://deb.debian.org/debian buster InRelease
		Get:2 http://security.debian.org/debian-security buster/updates InRelease [65.4 kB]
		Get:3 http://deb.debian.org/debian buster-updates InRelease [51.9 kB]
		Get:4 http://security.debian.org/debian-security buster/updates/main Sources [180 kB]
		Get:5 http://security.debian.org/debian-security buster/updates/main amd64 Packages [271 kB]
		Get:6 http://security.debian.org/debian-security buster/updates/main Translation-en [145 kB]
		Fetched 714 kB in 0s (1,469 kB/s)                              
		Reading package lists... Done
		Building dependency tree       
		Reading state information... Done
		1 package can be upgraded. Run 'apt list --upgradable' to see it.
		root@deb51:~# apt list --upgradeable
		Listing... Done
		libopenjp2-7/stable 2.3.0-2+deb10u2 amd64 [upgradable from: 2.3.0-2+deb10u1]
		N: There is 1 additional version. Please use the '-a' switch to see it
		```

	In the example, we see that the file libopenjp2-7 can be upgraded (slightly). That file is part of a package called OpenJPEG which deals with JPEG compression. This could be considered to be bloat (unnecessary software) on a server, especially if we are working in the command line only with no desktop environment (which you normally should be if working on Debian server). 

	> Note:
		This is just an example. Your server will have different information based on what is installed, and when you run the commands!

	In this example, we have a several options. We could:

	- Do nothing. This is, in fact, a common option.

	- Update the individual file (which is done with the apt install command)

		`apt install libopenjp2-7`

	- Update the whole system (which would only update that file anyway in this case)

		`apt upgrade`

	- Or, choose to remove the file or JPEG compression altogether in an effort to remove bloat. For example:

		`apt remove libopenjp2-7`

	If we choose one of the second or third options, the file gets updated, and a subsequent execution of `apt update` will result in a message telling us that "all packages are up to date". If we choose the last option, the file and other dependent files are removed, and a subsequent `apt update` will, again, show that all packages are up to date. 	
- Update another package that was previously installed

	For example, update the OpenSSH pacakge:

	`apt install openssh-server`

	That updates the SSH server on the system (if an update is available).

- Clean up!

	This is fairly easy. First, delete extra files in the /var directory with this command:

	`apt clean`

	Next, remove any unused or unnecessary packages:

	`apt autoremove`

	This will most likely remove a lot of unused pacakges. When it is finished, run an `apt udpate` to make sure that there are no additional updates that are required. 

Lastly, you should know how to update the OS from one point release to the next and from one version to the next. 

- To update the point release, first check the current version and point release. Here's an example in Debian:

	```
	root@deb51:~# cat /etc/debian_version 
	10.9
	```

	The file "debian_version" simply tells us the version and release. But it is important because it shows that point release number, whereas commands such as `hostnamectl` will only show the main version number. 

	`apt update && apt upgrade` will normally update the system to the latest point release (for example, from 10.8 to 10.9). it will also install any and all security updates (which is what we are really interested in). 

- To update from one version to another, the command `apt full-upgrade` can be used. But use caution, this is a major upgrade, and as such, it could cause system instability based on your hardware and software. It is best to run this on a test system first and document the results carefully. 

That's it for this lab - and for this section!