# How to install DaVinci Resolve on CentOS

This is a guide on how to install  DaVinci resolve on a generic, upstream build of CentOS. Because software is constantly changing, document is hosted on GitHub Pages so that changes can be made as necessary.

These particular instructions are tailored to an HP Z8 G4 workstation with a GTX 1080 Ti installed.

1. UEFI
	1. Set to boot to USB drive first
	<!-- I'm not sure if disabling secure boot is necessary. Need to test. -->
	2. [Disable Secure Boot](https://access.redhat.com/solutions/3421621)
2. Install CentOS from USB
	1. Include only GNOME Desktop
	2. Set up DHCP
3. Reboot
4. CentOS's installation interacts with HP's UEFI in such a way as to change the boot order, so reboot, and you'll boot to the M.2 SSD with the fresh installation
	1. Reboot and you'll boot to the M.2 SSD with the fresh installation
	2. Accept the CentOS license
	3. You can then safely eject the USB installation disk
5. Install CentOS updates
6. Install NVIDIA drivers
	1. Install `gcc`
	
		```sudo yum install gcc```
	2. Install kernel headers
	
		```sudo yum install kernel-devel-$(uname -r) kernel-headers-$(uname -r)```
	3. Download the driver from [NVIDIA's site](http://www.nvidia.com/drivers)
	4. Blacklist the Nouveau driver:
		
		1. Create a new GRUB configuration with the blacklisted Nouveau driver:
			
			1. Edit `/etc/default/grub` Append the following to `GRUB_CMDLINE_LINUX`:	
			
				```rd.driver.blacklist=nouveau nouveau.modeset=0```
			
			2. The whole line should read:
				
				```GRUB_CMDLINE_LINUX=“crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet rd.driver.blacklist=nouveau nouveau.modeset=0”```
			
		2. Create a file at `/etc/modprobe.d/blacklist-nouveau.conf` with the following contents:
				
				```blacklist nouveau```
				```options nouveau modeset=0```
				
		3. Regenerate the kernel `initramfs`:
				
				```sudo mv /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r)-nouveau.img```
				```dracut /boot/initramfs-$(uname -r).img $(uname -r)```
				
		4. Install EPEL
				
				```sudo yum install epel-release```
			
		5. Install DKMS
				
				```sudo yum install dkms```
				
		6. Reboot
		7. Don't log into GNOME
		8. Switch over to the F2 virtual console: `Ctrl` + `Alt` + `Fn` + `F2`
		9. Log in
		10. Stop `gdm`
		
			```sudo service gdm stop```
			
			This might pop you back over to the F1 virtual console, where the GUI normally lives, not populated by a bunch of status indicators scattered across the screen. Pop back over to F2 again.
		
		11. Go to the `~/Downloads` folder, or wherever you put the NVIDIA driver
		12. Change permissions on the driver so that it’s exectuable:
		
			```chmod 755 <NVIDIA-Linux-x86_64-whateverversion>.run```
			
		13. Run it:
		
			```sudo ./chmod 755 <NVIDIA-Linux-x86_64-whateverversion>.run```
			
		14. Register the kernel module with DKMS, so that a new module can be built later if a different kernel is installed
		15. Install the 32-bit compatibility libraries--better safe than sorry
		16. Opt to have NVIDIA update the `xconfig` file
