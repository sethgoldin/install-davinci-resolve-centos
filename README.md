# How to install DaVinci Resolve on CentOS

Here are some notes on how to install  DaVinci Resolve on CentOS. Though Blackmagic Design distributes a build of CentOS that's pre-installed with DaVinci Resolve and some other software required for DaVinci Resolve to run, Blackmagic Design's build can't be used to create a bootable USB drive&mdash;[it can only be used to create a DVD](https://forum.blackmagicdesign.com/viewtopic.php?f=21&t=65447#p370722). Upstream builds of "generic" CentOS straight from [the CentOS project](https://www.centos.org/) suffer no such limitation.

Because software is constantly changing, [this document is hosted on GitHub Pages](https://github.com/sethgoldin/davinci-resolve-generic-centos). If you find something wrong or outdated, please do open a pull request. 

These particular notes were originally worked out from installations to multiple HP Z8 G4 workstations with GTX 1080 Ti cards installed, but the information should be useful for other x86_64 systems as well.

1. UEFI
	1. Set to boot to a USB drive first
	2. [Disable Secure Boot](https://access.redhat.com/solutions/3421621)
2. Install CentOS from USB
	1. Include only GNOME Desktop
	2. Set up DHCP
	3. It's possible that the CentOS installer will not show a mouse or will display windows strangely. It might be necessary to install via the "simple graphical interface" under the "rescue" GRUB option. Later, once system is installed with the GUI up and running, you'll want to [set GNOME to start automatically at boot](https://www.rootusers.com/how-to-start-gui-in-centos-7-linux/).
3. Reboot
4. CentOS's installation interacts with HP's UEFI in such a way as to change the boot order, so reboot, and you'll boot to the M.2 SSD with the fresh installation
	1. Reboot and you'll boot into the M.2 SSD with the fresh installation
	2. Accept the CentOS license
	3. You can then safely eject the USB installation disk
5. Install CentOS updates
6. Install NVIDIA drivers
	1. Install `gcc`
	
		```sudo yum install gcc```
		
		N.B you don't want to install "Development Tools" packages from the OS installation that might install a more modern version of `gcc`&mdash;then you'd have a version of `gcc` different than the one that was used to build the kernel you just installed, and you'd have to go set an environmental variable to change how the NVIDIA driver sees `gcc`. It gets messy. Just wait to install `gcc` from `yum` after the OS installation so that it matches whatever your kernel was compiled with.
		
	2. Install kernel headers
	
		```sudo yum install kernel-devel-<version>```
		
		Let bash automatically complete with the correct, current version,by hitting the `Tab` key.
		
	3. Download the driver from [NVIDIA's site](http://www.nvidia.com/drivers)
	4. Blacklist the Nouveau driver by creating a new GRUB configuration:
		
		1. Edit `/etc/default/grub` Append the following to `GRUB_CMDLINE_LINUX`:	
			
			```rd.driver.blacklist=nouveau nouveau.modeset=0```
			
		2. The whole line should read:
				
			```GRUB_CMDLINE_LINUX=“crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet rd.driver.blacklist=nouveau nouveau.modeset=0”```
			
		3. Create a file at `/etc/modprobe.d/blacklist-nouveau.conf` with the following contents:
				
			```blacklist nouveau```
		
			```options nouveau modeset=0```
			
		4. Back up and regenerate the kernel `initramfs`:
				
			```sudo mv /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r)-nouveau.img```
			
			```sudo dracut /boot/initramfs-$(uname -r).img $(uname -r)```
			
		5. Reboot
				
	5. Install EPEL
				
		```sudo yum install epel-release.noarch```
			
	6. Install DKMS
				
		```sudo yum install dkms```
				
	7. Reboot
	8. Don't log into GNOME
	9. Switch over to the F2 virtual console: `Ctrl` + `Alt` + `F2`
		1. You might have to also press the `Fn` key if you're using an Apple keyboard.
	10. Log in
	11. Stop `gdm`
	
		```sudo service gdm stop```
		
		This might pop you back over to the F1 virtual console, where the GUI normally lives, now populated by a bunch of status indicators scattered across the screen. Pop back over to F2 again.
		
	12. Go to the `~/Downloads` folder, or wherever you put the NVIDIA driver
	13. Change permissions on the driver so that it’s exectuable:
		
		```chmod 755 <NVIDIA-Linux-x86_64-whateverversion>.run```
			
	14. Run it:
		
		```sudo ./<NVIDIA-Linux-x86_64-whateverversion>.run```
			
		1. Register the kernel module with DKMS, so that a new module can be built later if a different kernel is installed
		2. Install the 32-bit compatibility libraries&mdash;better safe than sorry
		3. Opt to have NVIDIA update the `xconfig` file
	15. Reboot
7. Download and install the latest DeckLink driver
	1. Download the latest driver [from the Blackmagic Design website](https://www.blackmagicdesign.com/support/family/capture-and-playback)
	2. You need to now become the root user. Type:
		
		```su -```
		
		When prompted, please enter your 'root' user's password.
		
	3. If you already have an older DeckLink driver installed, uninstall it:
		
		```rpm -qa | grep desktopvideo | xargs rpm -e```
		
	4. Uncompress the downloaded driver package. Type:
		
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
		
		1. The installer might tell you that you need `libGLU.so.1`, so install:
				
			```sudo yum install libGLU```
		
	8. After the installation completes, you should see the terminal prompt. Reboot.
	9. After the machine has rebooted, open a Terminal shell again
	10. You need to now become the root user. Type:
		
		```su -```
		
		When prompted, please enter your 'root' user's password
		
	11. You might need to update the firmware on your DeckLink card. Type:
		
		```BlackmagicFirmwareUpdater update 0```
		
	12.  If a firmware update was applied, reboot the machine after it completes. If no firmware update was required, a reboot is not necessary.
	
8. Install DaVinci Resolve
	1. Download `DaVinci_Resolve_Studio_14.3_Linux.zip` (if you have a DaVinci Resolve license dongle) or `DaVinci_Resolve_14.3_Linux.zip` [from the Blackmagic Design website](https://www.blackmagicdesign.com/support/family/davinci-resolve-and-fusion).
	2. Open a Terminal shell
	3. You need to now become the root user. Type:

		```su -```
		
		When prompted, please enter your root user's password.
		
	4. Resolve might not launch--if you run it via the command-line from `/opt/resolve/bin/`, you can look for clues as to why it might not be able to launch. It might be complaining about `libpng12.so(1)`, so install:
		
		```sudo yum install libpng12```
