# How to install DaVinci Resolve on CentOS 7.7

1. Create a bootable USB drive
	1. On Windows:
		1. Download [DVD ISO](http://isoredirect.centos.org/centos/7/isos/x86_64/)
		1. [Verify the download](https://wiki.centos.org/TipsAndTricks/sha256sum)
		1. Download and use [Rufus](https://rufus.ie/) to create the bootable USB drive
	1. On Mac or Linux:
		1. Download [DVD ISO](http://isoredirect.centos.org/centos/7/isos/x86_64/)
		1. [Verify the download](https://wiki.centos.org/TipsAndTricks/sha256sum)
		1. Use `dd` to create the bootable USB drive		
1. UEFI settings
	1. Set to boot to a USB drive first
	1. Disable Secure Boot and disable Legacy BIOS mode
1. Install CentOS from USB
	1. Include only GNOME Desktop
	1. Set up DHCP
	1. Set password for `root` account and create just one administrator account
	1. It's possible that the CentOS installer will not show a mouse or will display windows strangely. It might be necessary to install via the "simple graphical interface" under the "rescue" GRUB option. Later, once system is installed with the GUI up and running, you'll want to [set GNOME to start automatically at boot](https://www.rootusers.com/how-to-start-gui-in-centos-7-linux/).
1. CentOS's installation interacts with HP's UEFI in such a way as to change the boot order
	1. Reboot to boot into the fresh installation
	1. Accept the CentOS license
	1. You can then safely eject the USB installation disk
1. Install CentOS updates and reboot

	```
	$ sudo yum update
	$ sudo reboot
	```
	
1. Install the [kernel source](https://wiki.centos.org/HowTos/I_need_the_Kernel_Source):

	```$ sudo yum install "kernel-devel-uname-r == $(uname -r)"```

1. Install EPEL

	```$ sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm```

1. Install DKMS
	
	```$ sudo yum install dkms```
	
1. Install ELRepo
	1. Import the GPG key:
		
		```$ sudo rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org```
		
	1. Install for CentOS 7:
	
		```$ sudo yum install https://www.elrepo.org/elrepo-release-7.0-4.el7.elrepo.noarch.rpm```

1. Installing EPEL should have downloaded and installed `gcc`, but just in case, make sure:

	```$ sudo yum install gcc```

1. Install NVIDIA driver from ElRepo:
	
	```$ sudo yum install kmod-nvidia.x86_64```
	
	The current version is `440.44-1.el7_7.elrepo`.
	
	For good measure, you'll probably want the `yum` plugin as well:
	
	```sudo yum install yum-plugin-elrepo```
	
	Then, reboot:
	
	```$ sudo reboot```
	
1. [OPTIONAL] Download and install the latest DeckLink driver

	1. Download the latest driver [from the Blackmagic Design website](https://www.blackmagicdesign.com/support/family/capture-and-playback)
	1. Become the `root` user:
		
		```$ su -```
		
		When prompted, enter your `root` user's password.
		
	1. If you already have an older DeckLink driver installed, uninstall it:
		
		```# rpm -qa | grep desktopvideo | xargs rpm -e```
		
	1. If GNOME didn't uncompress it for you already, uncompress the downloaded driver package:
		
		```# tar xvfz /path/to/downloaded/driver/location/Blackmagic_Desktop_Video_Linux_<driver_version>.tar.gz```
		
	1. `cd` into the `rpm` folder, since this is CentOS
	
		```# cd /Blackmagic_Desktop_Video_Linux_<driver_version>/rpm/<yourarchitecture>```
		
	1. Install the latest Desktop Video driver, GUI, and Media Express. Type:

		1. ```# rpm -ivh desktopvideo-<driver_version>.x86_64.rpm```

		1. ```# rpm -ivh desktopvideo-gui-<driver_version>.x86_64.rpm```
		
		1. ```# rpm -ivh mediaexpress-<version>.x86_64.rpm```
		
			1. The installer might fail and tell you that you `mediaexpress` needs `libGLU.so.1`, so install `libGLU` and try again:
				
				```# yum install mesa-libGLU```
		
	1. After the installation completes, you should see the terminal prompt. Reboot.
	1. After the machine has rebooted, open a Terminal shell again
	1. Become the `root` user again:
		
		```$ su -```
		
		When prompted, please enter your `root` user's password
		
	1. You might need to update the firmware on your DeckLink card. Type:
		
		```# BlackmagicFirmwareUpdater update 0```
		
	1.  If a firmware update was applied, reboot the machine after it completes. If no firmware update was required, a reboot is not necessary.
	
1. Now we should be totally ready for DaVinci Resolve.
	1. N.B. If you didn't already install `mesa-libGLU` for Media Express, Resolve definitely needs it, so make sure to install it:
		
		1. `$ sudo dnf install mesa-libGLU`
		
		1. Then, reboot.
	
1. Install DaVinci Resolve
	1. Download and extract `DaVinci_Resolve_Studio_16.1.2_Linux.zip` (if you have a DaVinci Resolve license dongle or key) or `DaVinci_Resolve_16.1.2_Linux.zip` [from the Blackmagic Design website](https://www.blackmagicdesign.com/support/family/davinci-resolve-and-fusion).
	1. Double-click the `.run` file to use the GUI installer
	1. Resolve might not launch after the installation--if you run it via the command-line from `/opt/resolve/bin/`, you can look for clues as to why it might not be able to launch. If some program is missing, try figuring out what Resolve needs and install via `yum`.
