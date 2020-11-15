## Check Partitions:

	lsblk

## Delete the partition in which Arch is to be installed: (If not clean installing then skip this)

	gdisk /dev/sda	#(Can be sdb, sdc, etc)

Then enter options:

	x
	z
	y
	y

## Preparing the partitions:

	cgdisk /dev/sda
	
	1. Boot Partition:	
		First Option: Blank Always
		Size: 1024MiB
		Hex Code: EF00
		Name: boot

	2. Swap Partition:
		Size: 8GiB	#(It's enough but can be increased if needed)
		Hex Code: 8200
		Name: swap
	
	3. Root Partition:
		Size: Blank	#(Rest of the space in the drive)
		Hex Code: Blank	
		Name: root

	Then select 'write' and then 'quit'.

## Making filesystems:

	1. mkfs.fat -F32 /dev/sda1
	
	2. mkswap /dev/sda2

	3. mkfs.ext4 /dev/sda3
		
## Mounting the partitions:

	1. mount /dev/sda3 /mnt
	
	2. mkdir /mnt/boot

	3. mount /dev/sda1 /mnt/boot

## Installing the base system:

	pacstrap -i /mnt base linux linux-firmware sudo nano

## Generating fstab:

	genfstab -U -p /mnt >> /mnt/etc/fstab

## Chroot:
	
	arch-chroot /mnt /bin/bash

## Setting Locale:

	1. nano /etc/locale.gen
	
	2. Press Ctrl+W and search for en_US.UTF-8. Do it again, remove # and exit nano
	
	3. locale-gen

	4. echo LANG=en_US.UTF-8 > /etc/locale.conf

	5. export LANG=en_US.UTF-8

## Set Time, Region and Hostname:

	1. ln -s /usr/share/zoneinfo/Asia/Kolkata > /etc/localtime	#(This is for me but the region can be changed after searching for it a bit)

	2. hwclock --systohc --utc

	3. echo host_name > /etc/hostname	#(Replace host_name with whatever hostname is desired)

	4. sudo systemctl enable fstrim.timer

## Enable some extra repositories (If you want to.):

	1. nano /etc/pacman.conf

	2. uncomment out the ones you like (For example: multilib, etc)

## Set Root password:

	1. passwd

	2. enter the root password

## Add user:

	useradd -m -g users -G wheel,storage,power -s /bin/bash user_name	#(Replace user_name with desired username)

## Setting user password:

	passwd user_name

## Setting sudo:

	1. EDITOR=nano visudo

	2. Press Ctrl+W, search for wheel, uncomment and add at the end: Defaults rootpw

## Install bash completion:

	pacman -S bash-completion

## Mount EFI:

	mount -t efivarfs efivarfs /sys/firmware/efi/efivars

## Install Bootloader (Grub):

	1. pacman -S grub efibootmgr intel-ucode
	
	2. mkdir /boot/efi

	3. mount /dev/sda1 /boot/efi

	4. lsblk (To check if it mounted correctly.)

	5. grub-install --target=x86_64-efi --bootloader-id=GRUB --efi-directory=/boot/efi --recheck

	6. grub-mkconfig -o /boot/grub/grub.cfg

###	Optional: Dual Boot with Windows 10

		1. pacman -S os-prober

		2. fdisk -l /dev (To check in which partition Windows boot EFI file is located. It should be located in a partition of size ~260MB called Windows EFI boot or EFI system or something)

		In my case the partition turned out to be /dev/nvme0n1p2 but it may be different for everyone.

		3. mount /dev/nvme0n1p2 /mnt

		4. os-prober

		5. Backup existing grub config: cp /boot/grub/grub.cfg /boot/grub/grub.cfg-origin

		6. grub-mkconfig -o /boot/grub/grub.cfg

## Set up Network Manager:

	1. pacman -S networkmanager

	2. systemctl enable NetworkManager.service

## Optional: Install Graphic Card drivers:

	First do an lspci to check for the Graphic cards attached.

	For Intel:

		1. pacman -S xf86-video-intel libgl mesa
	
	For NVIDIA:

		1. pacman -S nvidia nvidia-lts nvidia-libgl mesa

## Exit and reboot:

	1. exit

	2. umount -R /mnt

	3. reboot

## After rebooting Enter username and password and then install the basic necessities:

	sudo pacman -S dkms linux-headers xf86-input-libinput xorg-xinput mesa xorg-server xorg-apps xorg-xinit xorg-twm xorg-xclock xfce4-terminal plasma sddm ccache ntfs-3g

	#(Here plasma and sddm can be replaced with gnome and gdm for GNOME desktop environment)

## Enabling the display manager:

	sudo systemctl enable sddm.service 

## Now the installation is complete so sudo reboot

## All set to go. Install desired apps and themes along with yay.


		

