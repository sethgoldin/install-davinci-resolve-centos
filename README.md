# How to install DaVinci Resolve on CentOS

Here are some notes on how to install  DaVinci Resolve on CentOS 8.0. Because software is constantly changing, [this document is hosted on GitHub Pages](https://github.com/sethgoldin/install-davinci-resolve-centos). If you find something wrong or outdated, please do open a pull request.

These particular notes were originally worked out from an installation to an HP Z8 G4 workstation with a single GTX 1080 Ti card installed, but the information should be useful for other x86_64 systems as well.

1. Create a bootable USB drive
	1. On Windows:
		1. Download [DVD ISO](https://www.centos.org/download/)
		1. [Verify the download](https://wiki.centos.org/TipsAndTricks/sha256sum)
		1. Download and use [Rufus](https://rufus.ie/) to create the bootable USB drive
	1. On Mac or Linux:
		1. Download [DVD ISO](https://www.centos.org/download/)
		1. [Verify the download](https://wiki.centos.org/TipsAndTricks/sha256sum)
		1. Use `dd` to create the bootable USB drive		
1. UEFI settings
	1. Set to boot to a USB drive first
	1. Disable Secure Boot and disable Legacy BIOS mode
1. Install CentOS from USB
	1. Software selection should be `Workstation` with only `GNOME Applications` checked.
	1. Set up DHCP
	1. Set password for `root` account and create just one administrator account
1. CentOS's installation interacts with HP's UEFI in such a way as to change the boot order, so reboot, and you'll boot to the M.2 SSD with the fresh installation
	1. Reboot and you'll boot into the M.2 SSD with the fresh installation
	1. Accept the CentOS license
	1. You can then safely eject the USB installation disk
1. Install CentOS updates and reboot
1. Take note of the IP address on the LAN, because you might need to SSH into the machine later.
1. Install the [kernel source](https://wiki.centos.org/HowTos/I_need_the_Kernel_Source):
	
	```$ sudo yum install "kernel-devel-uname-r == $(uname -r)"```
1. Install [EPEL](https://fedoraproject.org/wiki/EPEL)
	
	```$ sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm```
1. Install DKMS
	
	```$ sudo yum install dkms```
1. Prepare for the NVIDIA driver
	1. Download [the `.run` file for 430.50 from NVIDIA's site](https://www.nvidia.com/Download/driverResults.aspx/151568/).
	
	1. Become the root user:
		
		```$ su -```
	1. Make the file executable:
		
		```# chmod +x NVIDIA-Linux-x86_64-430.50.run```
	1. Blacklist the nouveau module:
		
		```# echo 'blacklist nouveau' >> /etc/modprobe.d/blacklist.conf```
	1. Install dependencies:
		
		```
		# dnf groupinstall "Workstation" "base-x" "Legacy X Window System Compatibility" "Development Tools"
		# dnf install elfutils-libelf-devel "kernel-devel-uname-r == $(uname -r)"
		```
	1. Back up and rebuild your `initramfs`:
		
		```
		# mv /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r)-nouveau.img
		# dracut -f
		```
	1. Change the default runlevel:
		
		```# systemctl set-default multi-user.target```
	1. Reboot the system:
		
		```# reboot```

1. From the command-line, log into `root`, navigate to wherever you put the `.run` file, and then install the NVIDA driver:
	
	1. ```# ./NVIDIA-Linux-x86_64-430.50.run```
		
		1. Be sure to install to DKMS
	1. Test the new driver:
		
		```# systemctl isolate graphical.target```
	1. If the test is successful, correct your default runlevel:
		
		```# systemctl set-default graphical.target```
	1. Reboot:
		
		```# reboot```
	1. Confirm that you're running the NVIDIA driver at any time by running `$ nvidia-smi`
1. Download and install the latest DeckLink driver
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
			
			```# dnf install mesa-libGLU```
	1. After the installation completes, you should see the terminal prompt. Reboot.
	1. After the machine has rebooted, open a Terminal shell again
	1. Become the `root` user again:
		
		```$ su -```
		When prompted, please enter your `root` user's password
	1. You might need to update the firmware on your DeckLink card. Type:
		
		```# BlackmagicFirmwareUpdater update 0```
	1.  If a firmware update was applied, reboot the machine after it completes. If no firmware update was required, a reboot is not necessary.

1. At this point, installing Desktop Video to DKMS will probably have "broken" the `grub` configuration again. If you try to log into the GUI, the screen will just go black. So we'll need to go rebuild the `grub` configuration again.
	1. If you can get to a virtual console, log in. Otherwise, SSH into the `root` account from a different machine on the network.
	1. `vim` into `/etc/default/grub`
	1. For the `GRUB_CMDLINE_LINUX` line, remove `rhgb` and add `rd.driver.blacklist=nouveau"`, so that the whole line is:
		
		```GRUB_CMDLINE_LINUX="crashkernel=auto resume=/dev/mapper/cl-swap rd.lvm.lv=cl/root rd.lvm.lv=cl/swap quiet rd.driver.blacklist=nouveau"```
	1. Write and close: `:wq`
	1. Rebuild the grub configuration again:
	
		```# grub2-mkconfig -o /boot/efi/EFI/centos/grub.cfg```
	1. Reboot:
	
		```# reboot```
1. Now we should be totally ready for DaVinci Resolve.
1. Install DaVinci Resolve
	1. Download and extract `DaVinci_Resolve_Studio_16.1.1_Linux.zip` (if you have a DaVinci Resolve license dongle or key) or `DaVinci_Resolve_16.1.1_Linux.zip` [from the Blackmagic Design website](https://www.blackmagicdesign.com/support/family/davinci-resolve-and-fusion).
	1. Double-click the `.run` file to use the GUI installer
	1. Resolve might not launch after the installation--if you run it via the command-line from `/opt/resolve/bin/`, you can look for clues as to why it might not be able to launch. If some program is missing, try figuring out what Resolve needs and install via `dnf`.
