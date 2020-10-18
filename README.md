[![Discord](https://img.shields.io/discord/364844043886395392.svg?color=007c8d&labelColor=222222&logoColor=888888&label=Discord&logo=discord)](https://discord.gg/b9PBjrs "Hello!, Need help with my guide then ask here.")
[![Donate with Bitcoin](https://en.cryptobadges.io/badge/micro/3Hqd4mameE1GzDNrfj2V9KAWaL7sUxJtA8)](https://en.cryptobadges.io/donate/3Hqd4mameE1GzDNrfj2V9KAWaL7sUxJtA8)

(Works with Arch ISO Image build as of: 2020.10.01)

# Arch Linux with KDE Plasma Installation Guide (UEFI & MBR)

Hello everyone, This is my guide for installing minimal Arch Linux with KDE Plasma Desktop Environment. In this guide we will go step by step on how I install my Arch System and set everything up from scratch for a stable & healthy OS.
</br>

## Table of Contents
  * [**Let's Begin**](#lets-begin)
  * [**Disk Partitioning**](#preparing-the-disk-for-system)
    * [UEFI System](#for-uefi-system)
    * [MBR System](#for-mbr-system)
  * [**Base System Installation**](#base-system-installation)
    * [Update Mirrors](#update-mirrors-using-reflector)
    * [Base System](https://github.com/XxAcielxX/arch-plasma-install#install-base-system)
    * [Generate fstab](#generate-fstab)
  * [**Chroot**](#chroot)
    * [Swapfile (UEFI only)](#create-swapfile-uefi-only)
    * [Date & Time](#set-time--date)
    * [Language](#set-language)
    * [Hostname & Hosts](#set-hostname)
    * [Network Manager](#install--enable-networkmanager) 
    * [ROOT Password](#set-root-password) 
    * [GRUB Bootloader](#install-grub-bootloader) 
      * [UEFI System](#for-uefi-system-1) 
      * [MBR System](#for-mbr-system-1)
  * [**Boot Freshly Installed System**](#unplug-the-usb-stick-and-boot-into-your-freshly-installed-arch-system)
    * [Add User](#add-new-user) 
    * [Sudo Command](#allow-wheel-group-to-use-sudo-commands) 
  * [**User Login**](#login-as-user)
    * [Display Server & GPU Drivers](#xorg--gpu-drivers)
    * [Multilib Repository (32bit)](#enable-multilib-repo-optional)
    * [Display Manager (SDDM)](#install--enable-sddm)
    * [Desktop Environment (KDE Plasma)](#kde-plasma--applications)
    * [Audio Utilities & Bluetooth](#audio-utilities--bluetooth)
    * [Misc Applications](https://#my-required-applications)
  * [**The Conclusion**](#the-conclusion) 
  * [**Extras (optional)**](#extras-optional)
    * [Yay](#install-yay)
    * [Zsh](#install-zsh)
  * [**Theming & Customizations**](theming--customisations)
     * [Oh My Zsh & Powerlevel10k Theme](#install-oh-my-zsh)
  * [**Maintenance, Performance Tuning & Monitoring**](maintenance-performance-tuning--monitoring)
    * [Paccache](#paccache)
    * [Cockpit](#install-cockpit)
  * [**Changelog**](#changelog)
</br>

## Let's begin
- Grab the latest Arch Image ISO from https://www.archlinux.org/download/ and write it to an USB Stick.
- After the image is done writing, it's time to boot into the Arch Live Environment. First thing you do is:

### Load Keymaps (for non US ENG Keyboard Users only)
For a list of all the available keymaps, use the command:
```
localectl list-keymaps
```

To search for a keymap, use the following command, replacing `[search_term]` with the code for your language, country, or layout:
```
localectl list-keymaps | grep -i [search_term]
```

### Now Loadkeys
```
loadkeys [keymap]
```

### Check for Internet Connectivity
```
ping -t 4 google.com
```
- If you are connected through Ethernet, then your Internet will be working out of the box.
- If you are using Wi-Fi, then use `wifi-menu` to connect to your local network.
- If this step is successful then we will head to next one.

### Update system clock
```
timedatectl set-ntp true
```
</br>

## Preparing the Disk for System

### \*** WARNING ***</br>
> Be extremely careful when managing your disks, incase you delete your precious data then DON'T blame me.
> Disk partitioning type (use UEFI or MBR, go according to your system).

## For UEFI System

### Disk Partitioning (UEFI)
We are going to make two partitions on our HDD, `EFI BOOT & ROOT` using `gdisk`.
- If you have a brand new HDD or if no partition table is found, then create GPT Partition Table by pressing `g`.
```
gdisk /dev/[disk name]
```
- [disk name] = device to partition, find yours by running `lsblk` and replace in all the below instances.
- We will be using one partition for our `/`, `/boot` & `/home`. 

```
n = New Partition
simply press enter = 1st Partition
simply press enter = As First Sector
+512M = As Last sector (BOOT Partition Size)
ef00 = EFI Partition Type

n = New Partition again
simply press enter = 2nd Partition
simply press enter = As First Sector 
simply press enter = As Last sector [ROOT Partition Size (using the remaining disk space left)]
8300 or simply press enter = EXT4 ROOT Partition Type

w = write & exit
```
### Format Partitions (UEFI)
```
mkfs.fat -F32 /dev/[efi partition name]
mkfs.ext4 /dev/[root partiton name] 
```

### Mount Partitions (UEFI)
```
mount /dev/[root partition name] /mnt
mkdir /mnt/boot/efi
mount /dev/[efi partition name] /mnt/boot/efi
```
## For MBR System

### Disk Partitioning (MBR)
We are going to make two partitions on our HDD, `SWAP & ROOT` using `cfdisk`.
- If you have a brand new HDD or if no partition table is found, then create MSDos Partition Table by selecting `msdos`.
```
cfdisk /dev/[disk name]
```
- [disk name] = device to partition, find yours by running `lsblk` and replace in all the below instances.
- SWAP Partition should double the size of RAM available in your system.
- We will be using one partition for our `/`, `/boot` & `/home`.

### Format the Partition, Make SWAP & Mount ROOT (MBR)
##### Format ROOT Partition as EXT4
```
mkfs.ext4 /dev/[root partition name]
```
##### Make & Turn SWAP Partition on (MBR)
```
mkswap /dev/[swap partition name]
swapon /dev/[swap partition name]
```
#### Mount ROOT Partition (MBR)
```
mount /dev/[root partition name] /mnt
```
</br>

## Base System Installation

### Update Mirrors using Reflector
```
reflector --country County1,Country2 --age 12 --protocol https --sort rate --save /etc/pacman.d/mirrorlist
```
Replace Country1 & Country2 with country names near to you or with the one you're living in. Refer to https://wiki.archlinux.org/index.php/reflector for more info.

### Install base system
```
pacstrap /mnt base base-devel linux linux-firmware linux-headers nano intel-ucode reflector
```
- Replace `linux` with *linux-hardened*, *linux-lts* or *linux-zen* to install the kernel of your choice.
- Replace `linux-headers` with Kernel type type of your choice respectively (e.g if you installed `linux-zen` then you will need `linux-zen-headers`).
- Replace `nano` with editor of your choice (i.e `vim` or `vi`).
- Replace `intel-ucode` with `amd-ucode` if you are using an AMD Processor.

### Generate fstab
(use `-U` or `-L` to define by [UUID](https://wiki.archlinux.org/index.php/UUID) or labels, respectively)
```
genfstab -U /mnt >> /mnt/etc/fstab
```
Check the resulting `/mnt/etc/fstab` file, and edit it in case of errors. 
</br>

## Chroot

```
arch-chroot /mnt
```
### Create Swapfile (UEFI only)
```
dd if=/dev/zero of=/swapfile bs=1G count=2 status=progress
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
```
Replace the above 1G in `bs=1G` with the amount of RAM installed your system.

### Add Swapfile entery in your `/etc/fstab` file (UEFI only) 
```
/swapfile    none    swap    defaults    0 0
```
Insert the above line at the bottom of `/etc/fstab`.

### Set Time & Date
```
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
hwclock --systohc
```
Replace `Region` & `City` according to your Time zone. Refer to https://wiki.archlinux.org/index.php/installation_guide#Time_zone.

## Set Language
We will use `en_US.UTF-8` here but, if you want to set your language, replace `en_US.UTF-8` with yours.

#### Edit locale.gen
```
nano /etc/locale.gen
```
Uncomment the below line
```
#en_US.UTF-8 UTF-8
```
save & exit.

### Generate Locale
```
locale-gen
```

### Add LANG to locale.conf
```
echo LANG=en_US.UTF-8 > /etc/locale.conf
```

## Set Hostname

```
echo arch > /etc/hostname
```
Replace `arch` with hostname of your choice.

### Set Hosts
```
nano /etc/hosts
```
#### add these lines to it
```
127.0.0.1    localhost
::1          localhost
127. 0.1.1   arch.localdomain arch
```
Replace `arch` with hostname of your choice.
save & exit.

### Install & Enable NetworkManager
```
pacman -S networkmanager
systemctl enable NetworkManager
```

### Set ROOT Password
```
passwd
```

### Install GRUB Bootloader
```
pacman -S grub
```

#### For UEFI System
```
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
```

#### For MBR System
```
grub-install --target=i386-pc /dev/[disk name]
```

### Create Grub configuration file
```
grub-mkconfig -o /boot/grub/grub.cfg
```

### Final Step
```
exit 
umount -a
reboot
```
</br>

## Unplug the USB Stick and boot into your freshly installed Arch system

### Login as ROOT

### Add new User
```
useradd -mG wheel [username]
```
Replace `[username]` with your username of choice.

### Set User Password
```
passwd [username]
```

### Allow Wheel Group to use Sudo Commands
```
EDITOR=nano visudo
```

#### Find and uncomment the below line
```
#%wheel ALL=(ALL) ALL
```
save & exit.

### Logout ROOT
```
exit
```

## Login as USER

### Check for updates
```
sudo pacman -Syu
```

### Xorg & GPU Drivers
```
sudo pacman -S xorg xf86-video-[your gpu type]
```
- For Nvidia GPUs, type `xf86-video-nvidia`.
- For newer AMD GPUs, type `xf86-video-amdgpu`.
- For legacy Radeon GPUs like HD 7xxx Series & below, type `xf86-video-ati`.

### Enable Multilib Repo (optional)
multilib contains 32-bit software and libraries that can be used to run and build 32-bit applications on 64-bit installs (e.g. [Wine](https://www.winehq.org/), [Steam](https://store.steampowered.com/), etc).

Edit `/etc/pacman.conf` & uncomment the below section.
```
[multilib]
Include = /etc/pacman.d/mirrorlist
```

#### MESA Libraries (32bit)
This package is required by Steam if you play games using Vulkan Backend.
```
sudo pacman -S lib32-mesa
```

### Install & Enable SDDM
```
sudo pacman -S sddm
sudo systemctl enable sddm
```

### KDE Plasma & Applications
```
sudo pacman -S plasma-desktop konsole dolphin ark kwrite kcalc spectacle ksysguard krunner kscreen partitionmanager
```
Packages | Description
--------- | ----------
plasma-desktop | Minimal Plasma DE installation.
konsole | KDE Terminal.
dolphin | KDE default File Manager.
ark | Archiving Tool.
kwrite | Text Editor.
kcalc | Scientific Calculator.
spectacle | KDE screenshot capture utility.
ksysguard | KDE System Task Monitor.
krunner | KDE Quick drop-down desktop search.
kscreen | KDE Display Setting Manager.
partitionmanager | KDE Disk & Partion Manager.

### Audio Utilities & Bluetooth
```
sudo pacman -S alsa-utils bluez bluez-utils
```
Packages | Description
--------- | ----------
alsa-utils | This contains (among other utilities) the `alsamixer` and `amixer` utilities.
bluez | Provides the Bluetooth protocol stack.
bluez-utils | Provides the `bluetoothctl` utility.

### My Required Applications
```
sudo pacman -S firefox qbittorrent wget screen git neofetch
```
Packages | Description
--------- | ----------
firefox | Mozilla Firefox Web Browser.
qbittorrent | A BitTorrent Client based on QT.
wget | Wget is a free utility for non-interactive download of files from the Web.
screen | Is a full-screen window manager that multiplexes a physical terminal between several processes, typically interactive shells.
git | Github Utility Tools.
neofetch | Neofetch is a command-line system information tool.

### Final Reboot
```
reboot
```
</br>

### The Conclusion
Now everything is installed and after the final `reboot`, you will land in you GUI Login Screen ready to use your system. You can also do some extra steps mentioned below to further improve your experience. I'll recommend you to install `yay` & `paccache`.
- Yay will provide your packages from AUR (Arch User Repository), which are not available in the Official Repo.
- Paccache can be used clean pacman cached packages either manually or in an automated way.
</br>

## Extras (optional)

### Install [Yay](https://github.com/Jguer/yay)
Yet Another Yogurt - An AUR Helper.
```
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```

### Install [Zsh](https://wiki.archlinux.org/index.php/zsh/) 
Zsh is a powerful shell that operates as both an interactive shell and as a scripting language interpreter.
```
sudo pacman -S zsh
```
Read *[here](#install-oh-my-zsh)* for customisation & theming for Zsh.
</br>

## Theming & Customisations

### Install [Oh My Zsh](https://ohmyz.sh/) 
Oh My Zsh is an open source, community-driven framework for managing your Zsh configuration.
```
sh -c "$(wget https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
```
My favourite theme is Powerlevel10k (follow below for installation).
- You can visit [here](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes) to download theme of your choice.

#### Get [Powerlevel10k](https://github.com/romkatv/powerlevel10k/) Theme for Oh My Zsh
This is the theme I'll install to spice up my terminal experience.
```
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

#### Configuration
***For new users***, on the first run, Powerlevel10k configuration wizard will ask you a few questions and configure your prompt. If it doesn't trigger automatically, type `p10k configure`. Configuration wizard creates `~/.p10k.zsh` based on your preferences. Additional prompt customization can be done by editing this file. It has plenty of comments to help you navigate through configuration options.

</br>

## Maintenance, Performance Tuning & Monitoring

### Paccache
Pacman Cache Cleaner.

Install
```
sudo pacman -S pacman-contrib
```

To manually clean pacman cache, run
```
paccache -rk3
```
Where, *k* indicates to keep "num" of each package in the cache.

#### To automate paccache process

Create a file in `/etc/pacman.d/hooks`
```
sudo mkdir /etc/pacman.d/hooks
sudo nano /etc/pacman.d/hooks/clean_cache.hook
```

Add the following lines in it
```
[Trigger]
Operation = Upgrade
Operation = Install
Operation = Remove
Type = Package
Target = *
[Action]
Description = Cleaning pacman cache...
When = PostTransaction
Exec = /usr/bin/paccache -rk3
```
save & exit.

### Install [Cockpit](https://cockpit-project.org/)
A systemd web based user interface for Linux servers, Workstations and even Desktops. Can be used to monitor your system stats, performance and perform various settings including updating of your system.
```
sudo pacman -S cockpit
```

##### Enable Cockpit
```
sudo systemctl enable --now cockpit.socket
```
Now open your browser and point to it `your-machine-ip:9000` and login with ***root*** to get full access.

</br>

## Changelog

  * **2020-10-18**
    * Added *Audio Utilities & Bluetooth* Section.
    * Added `Zsh` along with `Oh My Zsh` & `Powerlevel10k` theme.
    * Added `cockpit` under *Maintenance, Performance Tuning & Monitoring* Section.
  * **2020-10-17**
    * Added `lib32-mesa` package under *Multilib Repo* Section.
    * Added Discord Badge.
    * Added Donate BTC Badge.
    * Improved Guide text formatting for easier reading.
  * **2020-10-16**
    * Initial guide created.
