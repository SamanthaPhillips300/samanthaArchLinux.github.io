# Arch Linux Installation Documentation

This is the process I went through to install Arch Linux in VMware Workstation.

Resources:


- https://wiki.archlinux.org/title/Installation_guide
- https://itsfoss.com/install-arch-linux/
- https://linuxhint.com/arch_linux_network_manager/


**IMPORTANT: Unless specified, use the default options when running commands**

## Pre-Installation

1. Downloaded the arch linux .iso file from a HTTPS Direct Downloads link found on the [Arch Linux Downloads](https://archlinux.org/download/) page. This is the link that I used: http://ca.us.mirror.archlinux-br.org/iso/2021.10.01/
2. Once I downloaded the "archlinux-2021.10.01-x86_64.iso" file, I used PowerShell to check the MD5 hash. There is a file called md5sums.txt located on the same page as the .iso file download that is used to compare the hash too.

	PowerShell Commands:
	1. Navigate to folder where the archlinux-2021.10.01-x86_64.iso file is located. (I placed mine in C:\Users\slp01\Documents\Arch)
	2. Run the following command to get the MD5 hash of the .iso file:
	
			get-filehash -Algorithm MD5 archlinux-2021.10.01-x86_64.iso
	
3. Result from the PowerShell command should equal that of the md5sums.txt file. If it doesn't the .iso file may be corrupted.

**Notes:**

I had a lot of trouble trying to verify the hash on a .iso file I downloaded, but after lots of trial and error (and downloading a different file) I was able to do it.

## Installation

#### 1. Setup the Virtual Machine in VMware

	1. Click on "File" in the top left corner and then select "New Virtual Machine" OR press Ctrl + N
	2. Select the "Typical (recommended) option
	3. Select the "Installer disc image file (iso):" option and browse to the location of the archlinux-2021.10.01-x86_64.iso file.
	4. Select "Linux" for the Guest operating system and "Other Linux 5.x and later kernel 64-bit" for the version.
	5. Name the Virtual Machine
	6. Make the Maximum disk size (GB) equal 20 and select the "Store virtual disk as a single file"
	7. Select Customize Hardware... and change the Memory to 2GB
	8. Finish
	
  
**BEFORE TURNING ON THE MACHINE DO THE FOLLOWING:**

1. Navigate to where your VM files are located and edit the .vmx file. (Mine was located here: C:\Users\slp01\Documents\Virtual Machines\Arch)
2. Insert the following two lines as lines 2 and 3 in the .vmx file and then save it:
	
		firmware="efi"
		bios.bootdelay=5000 
	
**Notes:**	
I added the bios.bootdelay=5000 to the .vmx file so when the VM turns on I would have longer to select the UEFI medium mode because I missed selecting it within the default timeframe a few times. 
	
	
#### 2. Power on the VM 
  
  After Powering on the VM, select the "Arch Linux install medium (x86_64, UEFI)" option

#### 3. Verify UEFI mode
 
 Once the root@archiso shows up to be able to type commands run the following command to make sure you are in UEFI mode. If there are no errors, it's good to continue.
	
    ls /sys/firmware/efi/efivars
	
#### 4. Update the System Clock

Check what the time is currently set to:

	timedatectl
  
List the available time zones to choose from (press q to exit the list):

    timedatectl list-timezones
 
Set the time zone (I chose America/Chicago):

	timedatectl set-timezone America/Chicago
  
Check time zone has been updated:

	timedatectl

#### 5. Partitioning

Check to see available hard drives:

      fdisk -l
  
 Partition the sda hard drive:
 
 1. Run the following commands to create an EFI system partition:
 
        fdisk /dev/sda
        Command (m for help): n
        Partition type: p
        Partition number: 1
        First sector: default option (2048)
        Last sector: +500M
        Command (m for help): t
        Hex code or alias (type L to list all): EF

 2. Run the following commands to create a root partition:
  
        Command (m for help): n
        Partition type: p
        Partition number: 2
        First sector: default option
        Last sector: default option
	
 3. Save and exit partitioning
	  
        Command (m for help): w

**Notes:**

      n = create new partition
      p = primary partition type
      t = change partition type
      EF = EFI system option
      w = write changes to disk and exit partitioning

The partitioning was quite challenging. The Arch Installation Guide on the Wiki was hard to understand, but I was able to find a different website (https://itsfoss.com/install-arch-linux/) that helped me understand the partitioning process better. One of the biggest struggles I had was trying to figure out how to do the sizing of each partition and setting the first partition to be an EFI system instead of a Linux system.

#### 6. Format Partitions

Once the partitions have been created, each newly created partition must be formatted with an appropriate file system.

**Note:** I used the file systems shown on the Arch Linux Install page and this website - https://itsfoss.com/install-arch-linux/ 

Run the following commands to format the partitions:


    mkfs.fat -F32 /dev/sda1
    mkfs.ext4 /dev/sda2
	
  
#### 7. Mount the file system

**Note:** Purpose of mounting the iso: To mount an ISO file means to access its contents as if it was recorded on a physical medium and then inserted in the optical drive.

Run the following command to mount the .iso file:

	  mount /dev/sda2 /mnt
	
#### 8. Install Essential Packages

**Note:** The Arch Linux Wiki Installation guide says to install the base, linux, and linux-firmware packages. These packages are essential for the system to run properly. I chose to add the man, sudo, vim, zsh, and base-devel packages. I chose to add the man, sudo, vim, and zsh because I knew I would want to use them later. I added the base-devel package on my second Arch Linux install because I had to install it later on when installing a package from the AUR on my first Arch installation. 

Run the following command to add the base, linux, linux-firmware, man, sudo, vim, zsh, and base-devel package:

	  pacstrap /mnt base linux linux-firmware man sudo vim zsh base-devel
	
#### 9. Configure the system 
	
  _1. Create an fstab file._
   
   **Note:** If you are like me you are probably thinking "What is this?", so here is the explanation I found on the Arch Wiki Install Guide:  The fstab file can be used to          define how disk partitions, various other block devices, or remote file systems should be mounted into the file system.
    
    
 
   Run the following command to create the fstab file:
   
   
	  genfstab -U /mnt >> /mnt/etc/fstab
   
   
   Run the following command to check the file has been created properly:
   
   
	  cat /mnt/etc/ftsab
	
  
  
  
  _2. Change Root_

  Run the following command to change root into the new system:
			     
           
     arch-chroot /mnt
	
  
 **Note:** Once again I was questioning what the purpose of this was, so here's a combination of explanations I found:
 A chroot is an operation that changes the apparent root directory for the current running process and their children. A program that is run in such a modified environment cannot access files and commands outside that environmental directory tree. Changing root is commonly done for performing system maintenance on systems where booting and/or logging in is no longer possible. By using arch-chroot you enter the mounted disk as root.
  
  
  _3. Time zone_
	
  Set the time zone by running the following two commands (I chose America/Chicago as the time zone):
  
  
	ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime
	hwclock --systohc
	
  
  
    
  _4. Localization_
	
  This is what sets the language, numbering, date and currency formats for your system.
  
  First, edit the file /etc/locale.gen and uncomment en_US.UTF-8 UTF-8:
  
  Run the following command:
  
  
	vim /etc/locale.gen
       
      
And then remove the # in front of en_US.UTF-8 UTF-8

**Note:**

	- Press "Insert" to be able to edit
	- Press "ESC" and type :wq to save the file and exit vim


  Next, run the following command to create the locale.conf file, and set the LANG variable: 


	 echo 'LANG=en_US.UTF-8' >> /etc/locale.conf
        
        
   Lastly, check that the locale.conf file was created correctly:
   
   
	 cat /etc/locale.conf
		
    
#### 10. Network Configuration

**Note:** I used "myarch" for the name of my host.

First, write the chosen hostname to the file /etc/hostname


	echo myarch > /etc/hostname
  
And then check to make sure it is correct:


	cat /etc/hostname
  
 Next, edit the /etc/hosts file
 
 Run the following command to be able to edit the file with vim:
 
 
	vim /etc/hosts
  

Then, do the following:


- Press "Insert" to begin typing
- Add the following lines to the file:
    
    
	  127.0.0.1		localhost
	  ::1				 localhost
	  127.0.1.1		myarch
        
        
- Press "ESC" and type :wq to save and exit vim
			
      
#### 12. Setup Network Manager

**Note:** This part gave me the most trouble. I had assumed that because I could ping 8.8.8.8 while setting up the installation that it meant I didn't have to do anything related to the network and that I would have an Internet connection once I booted into the system. I was mistaken. The first time I booted into the GUI I quickly realized I was unable to ping anything. Thankfully, I had a recent snapshot that I was able to revert back to and didn't have to go through the entire installation process again. Setting up the Network Configuration was really confusing to me on the Arch Linux Wiki Install, but I was able to find this website (https://linuxhint.com/arch_linux_network_manager/) that helped me set it up.

Run the following commands to install a Network Manager:


	pacman -Syu
	pacman -S wpa_supplicant wireless_tools networkmanager
  
  
      
#### 11. Change the root password

Run the following command to change the root password, and follow the prompts it gives:


	passwd
	
		

		
#### 13. Install Grub bootloader

**Note:** A bootloader provides an interface for the user to load an operating system and applications. I chose to use the Grub bootloader because that is what I found instructions for :) 

Run the following commands to install the Grub bootloader:


	pacman -S grub efibootmgr
	mkdir /boot/efi
	mount /dev/sda1 /boot/efi
	grub-install --target=x86_64-efi --bootloader-id=GRUB --efi-directory=/boot/efi
	grub-mkconfig -o /boot/grub/grub.cfg
		
       
#### 14. Install Gnome Desktop environment

**Note:** I chose to install the Gnome desktop environment. I chose this because one of the websites I found that helped me through the installation had instruction for installing it :)

Run the following two commands to install the Gnome desktop environment:


	pacman -S xorg
	pacman -S gnome
  
  
		
#### 15. Enable display manager and network manager

Run the following commands to enable the Display Manager and Network Manager:


	systemctl enable gdm.service
	systemctl enable NetworkManager.service
	systemctl enable wpa_supplicant.service
		
        
#### 16. Finish

And finally, run the "exit" command to exit out of chroot and then shutdown the system:

	exit
	shutdown now
  
  
After the VM shutdowns, turn it back on and boot into the GUI.

**Note:** Overall the process of installing Arch Linux was really challenging, but I did learn quite a bit. I found the Arch Linux Wiki to be extremely confusing, so finding other installation guides to go along with it really helped me understand what the commands were doing and how to properly use them.
