# Arch Linux Installation

## Verify internet connection with:
```
ping google.com
```

## Update system clock:
```
timedatectl set-ntp true
```

Verify service status with:
```
timedatectl status
```

## Partition disk
List available drives:
```
fdisk -l
```

Begin partitioning tables:
```
fdisk /dev/sda
```
Type `m` when help is needed.

Since we are doing a EFI partition, press `g` to create a GPT partition table.

Type `n` to add a new parition.

First parition number type `1`.

Next press enter to leave first sector as default.

This parition is our EFI Partition, so type `+500M` to make this partition 500 MB.

Type `n` to again to add another parition.

Type `2`.

Once again press enter to leave first sector as default.

This parition is for the Arch OS, so press enter to allocate the remainder of the disk.

We must change the partition type of the first partition.
1. Press `t`.
2. Press `1` to select the first partition.
3. Press `1` to change this partition type to an EFI System.

Now we can press `w` to write the table to the desk and exit fdisk.

## Format Partitions
To set up the file system for our EFI partition:
```
mkfs.fat -F32 /dev/sda1
```

For our root partition:
```
mkfs.ext4 /dev/sda2
```
Now we must mount our root partition:
```
mount /dev/sda2 /mnt
```

## Installation
To install the essential packages for Arch:
```
pacstrap /mnt base linux linux-firmware
```

## Configuring the System
Generate an fstab file:
```
genfstab -U /mnt >> /mnt/etc/fstab
```

Change into the root of the new Arch system:
```
arch-chroot /mnt
```
We are logged in as root on our root partition /sda2.
- Note: You must be in this directory for the remainder of this guide.

Find your timezone with:
```
ls /usr/share/zoneinfo/
```
Use this to find your REGION and CITY, and the set your timezone using:
```
ln -sf /usr/share/zoneinfo/REGION/CITY /etc/localtime
```
Set hardware clock:
```
hwclock --systohc
```

### Localization
Install nano:
```
pacman -S nano
```
Edit `/etc/locale.gen`:
```
nano /etc/locale.gen
```
Now uncomment your locale. For those in America, uncomment `en_US.UTF-8 UTF-8`. Save and exit the file.

Now run:
```
locale-gen
```
## Network Configuration
Create the hostname file and add a hostname:
```
nano /etc/hostname
hostname
```
Open the hosts file and add the following:
```
nano /etc/hosts
```
```
127.0.0.1   localhost
::1         localhost
127.0.1.1	myhostname.localdomain  myhostname
```
- myhostname = what you found previously in the hostname file

Save and close the file.

## Root Password
Set your root password:
```
passwd
```

## Install Boot Loader
For this guide we will be using GRUB.

Install GRUB and efibootmgr:
```
pacman -S grub efibootmgr
```
Mount the EFI system partition:
```
mkdir /boot/efi
mount /dev/sda1 /boot/efi
```
Install the GRUB EFI application and its modules:
```
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
```
Generate GRUB configuration file:
```
grub-mkconfig -o /boot/grub/grub.cfg
```

## Network Configuration
Install and enable DHCP:
```
pacman -S dhcpcd
systemctl enable dhcpcd
```
Install and enable NetworkManager
```
pacman -S networkmanager
systemctl enable NetworkManager
```

## Add sudo users
Create users using the following commands:
```
useradd -m username
passwd username
usermod -aG wheel username
```
Install sudo:
```
pacman -S sudo
```
Open the visudo file with the following command and uncomment the line that says `%wheel ALL=(ALL) ALL`.
```
EDITOR=nano visudo
```

If we want to make a user change their password after login:
```
passwd -e username
```

## Reboot System:
```
exit
umount -l /mnt
reboot
```

## Install Desktop Environment
For this installation we will install KDE Plasma.

Install the following packages:
```
sudo pacman -S xorg sddm plasma kde-applications
```
Enable the Display Manager and Network Manager services:
```
sudo systemctl enable sddm.service
sudo systemctl enable NetworkManager.service
```
Now reboot.

## Add Aliases:
Open the .bashrc file:
```
nano ~/.bashrc
```
Add the alias to the file and save. For this example, we will add an alias that shows important memory information.
```
alias meminfo='free -m -l -t'
```
Source the .bashrc file for the aliases to start working.
```
. ~/.bashrc
```

