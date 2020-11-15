1. Check Partitions:

	lsblk

2. Delete the partition in which Arch is to be installed: (If not clean installing then skip this)

	gdisk /dev/sda	#(Can be sdb, sdc, etc)

Then enter options:

	x
	z
	y
	y

3. Preparing the partitions:

	cgdisk /dev/sda
	
	a) Boot Partition:	
		First Option: Blank Always
		Size: 1024MiB
		Hex Code: EF00
		Name: boot

	b) Swap Partition:
		Size: 8GiB	#(It's enough but can be increased if needed)
		Hex Code: 8200
		Name: swap
	
	c) Root Partition:
		Size: Blank	#(Rest of the space in the drive)
		Hex Code: Blank	
		Name: root

	Then select 'write' and then 'quit'.

4. Making filesystems:

	a) mkfs.fat -F32 /dev/sda1
	
	b) mkswap /dev/sda2

	c) mkfs.ext4 /dev/sda3
		
5. Mounting the partitions:

	a) mount /dev/sda3 /mnt
	
	b) mkdir /mnt/boot

	c) mount /dev/sda1 /mnt/boot

6. Installing the base system:

	pacstrap -i /mnt base linux linux-firmware sudo nano

7. Generating fstab:

	genfstab -U -p /mnt >> /mnt/etc/fstab

8. Chroot:
	
	arch-chroot /mnt /bin/bash

9. Setting Locale:

	a) nano /etc/locale.gen
	
	b) Press Ctrl+W and search for en_US.UTF-8. Do it again, remove # and exit nano
	
	c) locale-gen

	d) echo LANG=en_US.UTF-8 > /etc/locale.conf

	e) export LANG=en_US.UTF-8

10. Set Time, Region and Hostname:

	a) ln -s /usr/share/zoneinfo/Asia/Kolkata > /etc/localtime	#(This is for me but the region can be changed after searching for it a bit)

	b) hwclock --systohc --utc

	c) echo host_name > /etc/hostname	#(Replace host_name with whatever hostname is desired)

	d) sudo systemctl enable fstrim.timer

11. Enable some extra repositories (If you want to.):

	a) nano /etc/pacman.conf

	b) uncomment out the ones you like (For example: multilib, etc)

12. Set Root password:

	a) passwd

	b) enter the root password

13. Add user:

	useradd -m -g users -G wheel,storage,power -s /bin/bash user_name	#(Replace user_name with desired username)

14. Setting user password:

	passwd user_name

15. Setting sudo:

	a) EDITOR=nano visudo

	b) Press Ctrl+W, search for wheel, uncomment and add at the end: Defaults rootpw

16. Install bash completion:

	pacman -S bash-completion

17. Mount EFI:

	mount -t efivarfs efivarfs /sys/firmware/efi/efivars

18. Install Bootloader (Grub):

	a) pacman -S grub efibootmgr intel-ucode
	
	b) mkdir /boot/efi

	c) mount /dev/sda1 /boot/efi

	d) lsblk (To check if it mounted correctly.)

	e) grub-install --target=x86_64-efi --bootloader-id=GRUB --efi-directory=/boot/efi --recheck

	f) grub-mkconfig -o /boot/grub/grub.cfg

	Optional: Dual Boot with Windows 10

	g) pacman -S os-prober

	h) fdisk -l /dev (To check in which partition Windows boot EFI file is located. It should be located in a partition of size ~260MB called Windows EFI boot or EFI system or something)

	In my case the partition turned out to be /dev/nvme0n1p2 but it may be different for everyone.

	i) mount /dev/nvme0n1p2 /mnt

	j) os-prober

	k) Backup existing grub config: cp /boot/grub/grub.cfg /boot/grub/grub.cfg-origin

	l) grub-mkconfig -o /boot/grub/grub.cfg

19. Set up Network Manager:

	a) pacman -S networkmanager

	b) systemctl enable NetworkManager.service

20. Optional: Install Graphic Card drivers:

	First do an lspci to check for the Graphic cards attached.

	For Intel:

		a) pacman -S xf86-video-intel libgl mesa
	
	For NVIDIA:

		a) pacman -S nvidia nvidia-lts nvidia-libgl mesa

21. Exit and reboot:

	a) exit

	b) umount -R /mnt

	c) reboot

22. After rebooting Enter username and password and then install the basic necessities:

	sudo pacman -S dkms linux-headers xf86-input-libinput xorg-xinput mesa xorg-server xorg-apps xorg-xinit xorg-twm xorg-xclock xfce4-terminal plasma sddm ccache ntfs-3g

	#(Here plasma and sddm can be replaced with gnome and gdm for GNOME desktop environment)

23. Enabling the display manager:

	sudo systemctl enable sddm.service 

24. Now the installation is complete so sudo reboot

25. All set to go. Install desired apps and themes along with yay.


		

