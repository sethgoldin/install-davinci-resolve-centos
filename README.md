# How to install DaVinci Resolve on CentOS

Here are some notes on how to install  DaVinci Resolve on CentOS. Though Blackmagic Design distributes a build of CentOS that's pre-installed with DaVinci Resolve and some other software required for DaVinci Resolve to run, Blackmagic Design's build can't be used to create a bootable USB drive&mdash;[it can only be used to create a DVD](https://forum.blackmagicdesign.com/viewtopic.php?f=21&t=65447#p370722). Upstream builds of "generic" CentOS straight from [the CentOS project](https://www.centos.org/) suffer no such limitation.

Because software is constantly changing, [this document is hosted on GitHub Pages](https://github.com/sethgoldin/davinci-resolve-generic-centos). If you find something wrong or outdated, please do open a pull request. 

These particular notes were originally worked out from installations to multiple HP Z8 G4 workstations with single GTX 1080 Ti cards installed, but the information should be useful for other x86_64 systems as well.

1. UEFI
	1. Set to boot to a USB drive first
	2. Disable Secure Boot and disable Legacy BIOS mode
2. Install CentOS from USB
	1. Include only GNOME Desktop
	2. Set up DHCP
	3. It's possible that the CentOS installer will not show a mouse or will display windows strangely. It might be necessary to install via the "simple graphical interface" under the "rescue" GRUB option. Later, once system is installed with the GUI up and running, you'll want to [set GNOME to start automatically at boot](https://www.rootusers.com/how-to-start-gui-in-centos-7-linux/).
3. CentOS's installation interacts with HP's UEFI in such a way as to change the boot order, so reboot, and you'll boot to the M.2 SSD with the fresh installation
	1. Reboot and you'll boot into the M.2 SSD with the fresh installation
	2. Accept the CentOS license
	3. You can then safely eject the USB installation disk
4. Install CentOS updates and reboot
5. Add kernel source, additional repositories, and [Dynamic Kernel Module Support](https://en.wikipedia.org/wiki/Dynamic_Kernel_Module_Support)
	1. Install the [kernel source](https://wiki.centos.org/HowTos/I_need_the_Kernel_Source)
	
		```sudo yum install kernel-devel```
			
	2. Install DKMS
		
		```sudo yum install dkms```
		
	3. Install ELRepo
	
		1. Import the GPG key:
		
			```sudo rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org```
		
		2. Install for CentOS 7:
		
			```sudo rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm```
		
	4. Install EPEL
	
		```sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm```
		
	5. Installing EPEL should have downloaded and installed `gcc`, but just in case, make sure:
	
		```sudo yum install gcc```

6. Install NVIDIA driver from ElRepo:
	
	```sudo yum install kmod-nvidia```
	
	Then, reboot.
	
7. Download and install the latest DeckLink driver

	1. Download the latest driver [from the Blackmagic Design website](https://www.blackmagicdesign.com/support/family/capture-and-playback)
	2. Become the `root` user:
		
		```su -```
		
		When prompted, enter your `root` user's password.
		
	3. If you already have an older DeckLink driver installed, uninstall it:
		
		```rpm -qa | grep desktopvideo | xargs rpm -e```
		
	4. If GNOME didn't uncompress it for you already, uncompress the downloaded driver package:
		
		```tar xvfz /path/to/downloaded/driver/location/Blackmagic_Desktop_Video_Linux_<driver_version>.tar.gz```
		
	5. `cd` into the `rpm` folder, since this is CentOS
	
		```cd /Blackmagic_Desktop_Video_Linux_<driver_version>/rpm/<yourarchitecture>```
		
	6. Change permissions on the files so that you can execute them:
	
		```chmod 755 desktopvideo-<version>.<architecture>.rpm```
		
		```chmod 755 desktopvideo-gui<version>.<architecture>.rpm```
		
		```chmod 755 mediaexpress-<version>.<architecture>.rpm```
		
	7. Install the latest Desktop Video driver, GUI, and Media Express. Type:

		```rpm -ivh desktopvideo-<driver_version>.x86_64.rpm```

		```rpm -ivh desktopvideo-gui-<driver_version>.x86_64.rpm```
		
		```rpm -ivh mediaexpress-<version>.x86_64.rpm```
		
		1. The installer might fail and tell you that you need `libGLU.so.1`, so install `libGLU` and try again:
				
			```sudo yum install libGLU```
		
	8. After the installation completes, you should see the terminal prompt. Reboot.
	9. After the machine has rebooted, open a Terminal shell again
	10. Become the `root` user again:
		
		```su -```
		
		When prompted, please enter your `root` user's password
		
	11. You might need to update the firmware on your DeckLink card. Type:
		
		```BlackmagicFirmwareUpdater update 0```
		
	12.  If a firmware update was applied, reboot the machine after it completes. If no firmware update was required, a reboot is not necessary.
	
9. Install DaVinci Resolve
	1. Download `DaVinci_Resolve_Studio_14.3_Linux.zip` (if you have a DaVinci Resolve license dongle) or `DaVinci_Resolve_14.3_Linux.zip` [from the Blackmagic Design website](https://www.blackmagicdesign.com/support/family/davinci-resolve-and-fusion).
	2. Install `libpng12`:
	
		```sudo yum install libpng12```
		
	3. Become the `root` user again. Type:

		```su -```
		
		When prompted, please enter your root user's password.
	
	4.	Navigate to the downloaded script and run it:
	
		```./DaVinci_Resolve_Studio_<version>_Linux.sh```
	
	5. Resolve might not launch after the installation--if you run it via the command-line from `/opt/resolve/bin/`, you can look for clues as to why it might not be able to launch. If some program is missing, try figuring out what Resolve needs and install via `yum`.
