# How to install DaVinci Resolve on CentOS 8.2

1. Create a bootable USB drive
	1. On Windows:
		1. Download [DVD ISO](https://www.centos.org/download/)
		1. [Verify the download](https://wiki.centos.org/TipsAndTricks/sha256sum)
		1. Download and use [Rufus](https://rufus.ie/) to create the bootable USB drive
	1. On Mac or Linux:
		1. Download [DVD ISO](https://www.centos.org/download/)
		1. [Verify the download](https://wiki.centos.org/TipsAndTricks/sha256sum)
		1. [Use `dd` to create the bootable USB drive](https://wiki.centos.org/HowTos/InstallFromUSBkey)
1. UEFI settings
	1. Set to boot to a USB drive first
	1. Disable Secure Boot and disable Legacy BIOS mode
1. Install CentOS from USB
	1. Software selection should be `Workstation` with only `GNOME Applications` checked.
	1. Set up DHCP
	1. Set password for `root` account and create just one administrator account
	1. Especially with a GeForce card, it's possible that the default CentOS installer will not show the graphics correctly. It might be necessary to install via the ["basic graphics mode" from the Troubleshooting menu in the boot options](https://docs.centos.org/en-US/centos/install-guide/Trouble-x86/#_problems_with_booting_into_the_graphical_installation).
1. CentOS's installation interacts with HP's UEFI in such a way as to change the boot order
	1. Reboot to the fresh installation of CentOS
	1. Accept the CentOS license
	1. You can then safely eject the USB installation disk
1. Install CentOS updates and reboot:
	
	```
	$ sudo dnf update --refresh
	$ sudo reboot
	```	
1. Install the [kernel source](https://wiki.centos.org/HowTos/I_need_the_Kernel_Source):
	
	```$ sudo dnf install "kernel-devel-uname-r == $(uname -r)"```
1. Install [EPEL](https://fedoraproject.org/wiki/EPEL)
	
	```$ sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm```
1. Install DKMS
	
	```$ sudo dnf install dkms```
1. Install ELRepo
	1. Import the GPG key:
		
		```$ sudo rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org```
		
	1. Install for CentOS 8:
	
		```$ sudo dnf install https://www.elrepo.org/elrepo-release-8.el8.elrepo.noarch.rpm```

1. Installing EPEL should have downloaded and installed `gcc`, but just in case, make sure:

	```$ sudo dnf install gcc```

1. Install NVIDIA driver from ELRepo:
	1. Install the NVIDIA driver:
	
		```$ sudo dnf install kmod-nvidia.x86_64```
	
		The current version is `450.57-1.el8_2.elrepo`.
		
	1. Then, reboot:
	
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
		
1. [OPTIONAL] Install Java if you want to perform Photon validation of IMF packages:

	```$ sudo dnf install java```
	
	
1. [OPTIONAL] If you want to use your workstation as a PostgreSQL client for collaborative workflows, and the network is either air-gapped or has a trustworthy network-wide firewall, you'll want to disable the individual firewall on the workstation so that the east-west traffic between workstations will function properly: for bin locking, timeline locking, collaborative chat, etc.

	```
	$ sudo systemctl stop firewalld
	$ sudo systemctl disable firewalld
	```
		
1. Install DaVinci Resolve
	1. Download and extract `DaVinci_Resolve_Studio_16.2.1_Linux.zip` (if you have a DaVinci Resolve license dongle or key) or `DaVinci_Resolve_16.2.1_Linux.zip` [from the Blackmagic Design website](https://www.blackmagicdesign.com/support/family/davinci-resolve-and-fusion).
	1. Double-click the `.run` file to use the GUI installer
	1. Resolve might not launch after the installation--if you run it via the command-line from `/opt/resolve/bin/`, you can look for clues as to why it might not be able to launch. If some program is missing, try figuring out what Resolve needs and install via `dnf`.
