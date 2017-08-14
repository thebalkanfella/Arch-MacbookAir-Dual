
# Installing Arch Linux on MacBook Air (13-inch, Mid 2013)

The following is my dual OSX - Arch linux set up:

## Configuration PHASES

#### 1. Boot into arch and make the harddisk ready, root with password. 1st reboot
#### 2. Boot into OSX and create boot file structure, copy boot.efi 2nd reboot
#### 3. Boot into arch and login as root and make new user. 3rd reboot
#### 4. Boot into arch,login as user and install your desktop environment.

### Create USBs for installation

#### Make a bootable USB with arch Installallation
#### Make a bootable USB with GParted (optional))

## PHASE 1 "Boot into arch and make the harddisk ready, root has a password. 1st reboot"

Use the latest Arch usb to boot from it

Use the US keyboard otherwise you have to select your keymap manually on “/usr/share /kbd/keymaps” directory

```
loadkeys <yours>
```
### Create partitions for installation

#### Use Gparted to format partitions after creating them on OSX optional or use fdisk, cgdisk. 
#### Create a SHARE partition to access from both systems at sda4.

```mkfs.ext4 /dev/sda5         (Arch bootloader) 128 MiB for Apple HFS+ 
mkfs.ext2 /dev/sda6         (boot linux)256 MiBmkswap /dev/sda7            (swap)	2 Gigmkfs.ext4 /dev/sda8         (root)	15 Gigmkfs.ext4 /dev/sda9         (home)	23 Gig
```
### Mount partitions 

```mount /dev/sda8 /mntmkdir /mnt/boot && mount /dev/sda6 /mnt/bootmkdir /mnt/home && mount /dev/sda9 /mnt/homeswapon /dev/sda7
```

### Network
Plugin ethernet cable via adapter.
It usually connects automatically. If necessary run the following: 

```
wifi-menu
```
To check connection:

```
ping yahoo.com
```

### Installation

```
pacstrap /mnt base base-devel
genfstab -p /mnt >> /mnt/etc/fstab
```

### Optimize parameters for SSD, open your fstab file:

```
nano /mnt/etc/fstab
```
Make the following changes for SSD:

```
/dev/sda6      /boot  ext2  defaults,relatime,stripe=4    0 2
/dev/sda8        /      ext4  defaults,noatime,discard,data=writeback   0 1
/dev/mapper/home /home  ext4  defaults,noatime,discard,data=ordered 0 2
/swapfile none  swap defaults                                0 0
```

### Pacman update

```
pacman -Sy
nano /etc/pacman.conf
```
un-comment “multilib” repo:

```
[multilib]
Include = /etc/pacman.d/mirrorlist
```

Add Yaourt repository at the bottom:

```
[archlinuxfr]
siglevel = Never
Server = http://repo.archlinux.fr/$arch
```

### Configure your new base system

```
arch-chroot /mnt /bin/bash 
```

Set up locale

```
sudo nano /etc/locale.gen
```
Uncomment desired locals
```
en_US.UTF-8 UTF-8 
en_US ISO-8859-1
```

Generate locales
```
locale-gen
```

### create a settings file to load default language on boot
```
echo LANG=en_US.UTF-8 > /etc/locale.conf
export LANG=en_US.UTF-8
```

### Keymap for the new installation

Check console fonts at https://alexandre.deverteuil.net/pages/consolefonts/

```
nano /etc/vconsole.conf
```
Insert/edit reqyuired keymap:
```
KEYMAP=us
FONT=lat9w-16
```
Ensure you've chosen the correct keymap otherwise you might be in trouble when login the first time.

### Link the preferred time zone to your localtime link:

```
rm /etc/localtime
ln -s /usr/share/zoneinfo/America/New_York /etc/localtime
```

### Enable the hardware clock

```
hwclock --systohc --utc
```

### Name the system
```echo ArchZain > /etc/hostname
```
```
nano /etc/hosts
```
Add the new hostname lines

```   127.0.0.1 localhost.localdomain localhost ArchZain   ::1 localhost.localdomain localhost ArchZain
```

### Network Manager 
```
pacman -S networkmanager
systemctl enable NetworkManager
```

### Edit mkinitcpio

```
nano /etc/mkinitcpio.conf
```

Insert “keyboard” after HOOKS as necessary:

```
HOOKS="base udev autodetect modconf block keyboard zfs filesystems"
```
Run mkinitcpio and set root password:

```
mkinitcpio -p linux
passwd
```

### Bootloader

```
pacman -S grub-efi-x86_64
```
Edit Grub 

```
nano /etc/default/grub 
```
Change it as follows:

```
GRUB_CMDLINE_LINUX_DEFAULT="quiet rootflags=data=writeback libata.force=1:noncq"
```
Also fix submenu

```
GRUB_DISABLE_SUBMENU=y
```

### Generate boot.efi

```
grub-mkconfig -o boot/grub/grub.cfg
```
```
grub-mkstandalone -o boot.efi -d usr/lib/grub/x86_64-efi -O x86_64-efi --compress=xz boot/grub/grub.cfg
```

### Copy boot.EFI to USB

Insert USB and list partitions 

```
fdisk -l```
Now copy boot.efi to the USB drive 
```mkdir /mnt/usbdisk && mount /dev/sdc1 /mnt/usbdiskcp boot.efi /mnt/usbdiskumount /dev/sdc1
```
Unmount your partitions

```
umount /mnt/boot 
umount /mnt/home 
swapoff /dev/sda7 
umount /mnt
```
### exit chroot:
```exitreboot
```

## PHASE 2 "Boot into OSX and create boot file structure, paste boot.efi 2nd reboot"
Power and press option key to boot in OSXOpen Disk Utility and format (“Erase”) /dev/disk0s5 ([128MB] Apple HFS+ “Boot Loader”)using Mac journaled filesystem

### Create boot file structure
Open terminal to set Arch in apple bootloader as default boot option.

```
cd /Volumes/ARCH\ BOOT
mkdir System mach_kernel
cd System
mkdir Library
cd Library
mkdir CoreServices
cd CoreServices
touch SystemVersion.plist
nano SystemVersion.plist
```

```
<xml version="1.0" encoding="utf-8"?>
<plist version="1.0">
<dict>
    <key>ProductBuildVersion</key>
    <string></string>
    <key>ProductName</key>
    <string>Linux</string>
    <key>ProductVersion</key>
    <string>Arch Linux</string>
</dict>
</plist>
```

### Copy boot.efi from your USB stick to the CoreServices directory.

User terminal or finder to copy boot.efi

### Make Boot Loader partition bootable

Disable the System Integrity Projection of OS X before proceeding:
Restart the computer, while booting hold down Command-R to boot into recovery mode.
Navigate to the “Utilities > Terminal” in the top menu bar.

```
csrutil disable
```
Restart option key in OSX open terminal and type the following:

```
sudo bless --device /dev/disk0s5 --setBoot
```

Reboot option key and select Arch linux to boot.## PHASE 3 "Boot into arch and login as root and make new user, 3rd reboot"login using root password

### Add new user and password

```
useradd -m -g users -G wheel,storage,power -s /bin/bash zain
passwd zain
```

### Check internet connection

```
ping yahoo.com
```
If not connected do the following otherwise skip this step:

```ip link
systemctl enable dhcpcd@ens9
ip link set ens9 up```

### Sudo rights

```
pacman -S sudo
EDITOR=nano visudo
```
Unckeck as follows:
```
%wheel ALL=(ALL) ALL
```

Bash completion
```
pacman -S bash-completion
```

Exit root

```
exit
```

## PHASE 4 "Boot into arch, login with username and install your desktop environment"

Check I3 installation and optimization for MacBook Air.


### Source of information without any particular order:

* [How to Dual-Boot OS X and Ubuntu by Nailen Matschke](http://courses.cms.caltech.edu/cs171/materials/pdfs/How_to_Dual-Boot_OSX_and_Ubuntu.pdf)
* [Arch Linux on a MacBook Pro](https://zanshin.net/2015/02/05/arch-linux-on-a-macbook-pro-part-1-creating-a-usb-installer/)
* [Arch Linux Installation with OS X on Macbook Air dual boot](http://panks.me/posts/2013/06/arch-linux-installation-with-os-x-on-macbook-air-dual-boot/)
* [pandeiro arch-on-air](https://github.com/pandeiro/arch-on-air)

