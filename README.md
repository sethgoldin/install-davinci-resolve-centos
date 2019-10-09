# How to install DaVinci Resolve on CentOS

Here are some notes on how to install  DaVinci Resolve on CentOS 8.0. Though Blackmagic Design distributes a build of CentOS 7.3 that's pre-installed with DaVinci Resolve and some other software required for DaVinci Resolve to run, Blackmagic Design's build can't be used to create a bootable USB drive&mdash;[it can only be used to create a DVD](https://forum.blackmagicdesign.com/viewtopic.php?f=21&t=65447#p370722). Upstream builds of "generic" CentOS straight from [the CentOS project](https://www.centos.org/) suffer no such limitation.

Because software is constantly changing, [this document is hosted on GitHub Pages](https://github.com/sethgoldin/install-davinci-resolve-centos). If you find something wrong or outdated, please do open a pull request. 

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
1. Install the [kernel source](https://wiki.centos.org/HowTos/I_need_the_Kernel_Source):
	
	```$ sudo yum install "kernel-devel-uname-r == $(uname -r)"```
1. Install [EPEL](https://fedoraproject.org/wiki/EPEL)
	
	```$ sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm```
1. Install DKMS
	
	```$ sudo yum install dkms```
1. Prepare for the NVIDIA driver
	1. Download [the `.run` file for 430.50 from NVIDIA's site](https://www.nvidia.com/Download/driverResults.aspx/151568/) and place it in `root`'s home directory.
	
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

1. From the command-line, log into `root` and then install the NVIDA driver:
	
	1. ```# ./NVIDIA-Linux-x86_64-430.50.run```
		
		1. Be sure to install to DKMS
	1. Test the new driver:
		
		```# systemctl isolate graphical.target```
	1. If the test is successful, correct your default runlevel:
		
		```# systemctl set-default graphical.target```
	1. Reboot:
		
		```# reboot```
	1. Switch to a virtual terminal and log into `root`.
	1. Remove `nomodeset` from the kernel cmdline in `/etc/default/grub`.
		1. `vim` into `/etc/default/grub`
		1. Remove `nomodeset`
		1. Write and close the file: `:wq`
	1. Rebuild the `grub` configuration:
		
		```# grub2-mkconfig -o /boot/efi/EFI/redhat/grub.cfg```
	1. Reboot:
		
		```# reboot```
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
			
			```# yum install mesa-libGLU```
	1. After the installation completes, you should see the terminal prompt. Reboot.
	1. After the machine has rebooted, open a Terminal shell again
	1. Become the `root` user again:
		
		```$ su -```
		When prompted, please enter your `root` user's password
	1. You might need to update the firmware on your DeckLink card. Type:
		
		```# BlackmagicFirmwareUpdater update 0```
	1.  If a firmware update was applied, reboot the machine after it completes. If no firmware update was required, a reboot is not necessary.
1. Install DaVinci Resolve
	1. Download and extract `DaVinci_Resolve_Studio_16.0_Linux.zip` (if you have a DaVinci Resolve license dongle or key) or `DaVinci_Resolve_16.0_Linux.zip` [from the Blackmagic Design website](https://www.blackmagicdesign.com/support/family/davinci-resolve-and-fusion).
	1. Double-click the `.run` file to use the GUI installer
	1. Resolve might not launch after the installation--if you run it via the command-line from `/opt/resolve/bin/`, you can look for clues as to why it might not be able to launch. If some program is missing, try figuring out what Resolve needs and install via `yum`.
1. [OPTIONAL] Install and configure PostgreSQL 9.5. If you just want to use a local disk database, you're all set. However, if you want use a local PostgreSQL database, or if you want to use this particular workstation as a PostgreSQL server for other Resolve workstations on your network, follow the additional steps below.
	1. Install the repository RPM:
		
		```$ sudo yum install yum install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm```
	1. Install the client packages:
		
		```$ sudo yum install postgresql95```
	1. Install the server packages:
		
		```$ sudo yum install postgresql95-server```
	1. Initialize the database and enable automatic start:
		
		```
		$ sudo /usr/pgsql-9.5/bin/postgresql95-setup initdb
		$ sudo systemctl enable postgresql-9.5
		$ sudo systemctl start postgresql-9.5
		```
	1. Set up a static IP address:
		1. Run either `$ ifconfig a` or `$ ip a`to view the name of the connected ethernet device.
		1. `vi` into `/etc/sysconfig/network-scripts/ifcfg-<yourinterfacename>`.
		1. Modify the specific parameters:
		
			```
			BOOTPROTO=none

			# Here, after the =, you can enter whatever you had previously with DHCP, since we know that the address assigned via DHCP worked
			IPADDR=<yourIPaddress>
			
			# Here's the subnet mask. Mine is 255.255.255.0, but your network might be different
			NETMASK=255.255.255.0
			
			# This one is going to depend on your particular router. Check your router
			GATEWAY=<yourgatewayIPaddress>

			# This will depend on your ISP. Since mine is Comcast, I use 75.75.75.75
			DNS1=75.75.75.75

			DEFROUTE=yes

			IPV4_FAILURE_FATAL=no

			# Disable IPv6
			IPV6INIT=no

			# Activate on Boot
			ONBOOT=yes
			```
		1. Restart the network service for these changes to take effect.
			
			```$ sudo systemctl restart network```
		1. Configure PostgreSQL for sharing by configuring the `pg_hba.conf` file:
			1. Become the `postgres` superuser:
				
				```$ sudo su - postgres```
			1. From `/var/lib/pgsql/`, `cd` into `/var/lib/pgsql/9.5/data`:
				
				```# cd 9.5/data/```
				If you check `# pwd`, you'll see that you're in `/var/lib/pgsql/9.5/data`.
			1. Make a copy of `pg_hba.conf`, just in case anything goes wrong:
				
				```# cp pg_hba.conf pg_hba.conf.backup```
			1. `vi` into `pg_hba.conf` and [modify it so as to enable sharing](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-centos-7) by adding in this line to the very bottom of the file:
				
				```host     all     all     <your workstation's static IP>/24     md5```
		1. Modify `postgresql.conf` to allow incoming TCP/IP sockets:
			1. `vi` into `/var/lib/pgsql/9.5/data/postgresql.conf`
			1. Scroll all the way to the bottom, and add the uncommented line:
				
				```listen_addresses = '*'```
		1. Assign a default password of `DaVinci` to the `postgres` user account
			1. Enter the `psql` shell:
				
				```# psql```
			1. Create a password for the user `postgres` by entering:
				
				```\password```
			1. Enter `DaVinci`. The `psql` shell will prompt you to reenter the password to confirm that you’ve typed it correctly.
			1. You can then exit the `psql` shell by entering `\q`.
			1. You can then exit from being the `postgres` superuser and get back to your regular user account by entering `exit`.
		1. Disable and stop the default CentOS firewall:
		
			```
			$ sudo systemctl disable firewalld
			$ sudo systemctl stop firewalld
			```
			Reboot the system.
		1. Verify that the PostgreSQL server is running properly:
			1. Enter:
				
				```cat /etc/services | grep 5432```
			You should see:
				
				```
				postgres     5432/tcp     postgresql     # POSTGRES
				postgres     5432/udp     postgresql     # POSTGRES
				```
			This means that PostgreSQL is good to go through port 5432.
		1. You can also check:
			
			```$ netstat -tulpn | grep 5432```
			Which should show that `0.0.0.0:5432` is good to go for TCP.
		1. Enter:
			
			```$ service postgresql-9.5 status```
			Here you’ll see some information about how TCP connections are listening through port 5432, with IPv4 IP addresses:
				
				```tcp     0     0 0.0.0.0:5432     0.0.0.0:*     LISTEN```		
	1. If you're going to be performing serious work with PostgreSQL databases, make sure that you back up and optimize the databases regularly, which can be done [automatically](https://github.com/sethgoldin/davinci-resolve-postgresql-workflow-tools).
