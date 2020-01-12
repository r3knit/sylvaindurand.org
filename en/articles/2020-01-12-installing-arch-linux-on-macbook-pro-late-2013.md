---
title: Arch Linux on Macbook Pro late 2013
alt: <em>Arch Linux</em> on <br>Macbook Pro late 2013
categories: Linux
---

Arch Linux is a distribution whose installation procedure may seem demanding, but which in return offers complete control over what is installed, but above all allows you to experiment with and understand many of the basic elements of how a Linux system works.

This article details the steps that allow me to get a minimum functional installation on a 13" MacBook Pro, late 2013 (`Macbook Pro 11.1`). This one is characterized by a few specificities, notably a recalcitrant *Broadcom* wifi card.

*It is strictly discouraged to follow these commands blindly*, because Arch Linux is a distribution that is very regularly updated and because each system is different.

Start by opening and following the [installation guide](https://wiki.archlinux.org/index.php/installation_guide) and the [MacbookPro11,x](https://wiki.archlinux.org/index.php/MacBookPro11,x) page of the [official wiki](https://wiki.archlinux.org/), which are very detailed. This article can serve as a complement to these two pages.


## Preparation of the installation medium

Let's start by downloading the Arch Linux installation image from the [official website](https://www.archlinux.org/download/). Here we will use a USB key to proceed with the installation. Warning: all its contents will be removed!


### On macOS

To know the name of the key, we use :

```bash
diskutil list
```

We then unmount the key, and proceed to copy the image (where `diskx` is to be modified by the name of your key) :

```bash
sudo diskutil unmountdisk /dev/diskx
sudo dd bs=1m if=archlinux_image.iso of=/dev/rdiskx conv=sync
```

### On Linux

To know the name of our USB key, we can use :

```bash
fdisk -l
```

To copy the image to the key, you can use the following command (where `sdx` is to be replaced by the name of your key). We restart by pressing `alt`.

```bash
dd bs=4M if=archlinux_image.iso of=/dev/sdx status=progress oflag=sync
```

## Installation

We can now restart the computer from the USB key, by pressing the *alt* key at boot (after the Mac initialization sound). We choose the `EFI Boot` menu, then `Arch Linux archiso x86_UEFI CD`. This brings us to the installer.


### Keyboard layout

By default, the english layout is used. If you need another layout, you can use:

```bash
loadkeys fr-latin1
```

### Display

Due to the HiDPI (Retina) screen, the entries are very small. It is possible for us to get a larger font for this terminal with :

```bash
setfont sun12x22
```

### Wifi

This is the most blocking part compared to the usual computers. The MacBook Pro for late 2013 usually comes with a *Broadcom* wifi card, which is known to be difficult to use with regular drivers.

The purpose of this part is only to have a network connection to do the installation, we are not yet configuring the wifi for our final system. If your system is already installed and you want to have a working wifi, you can go further down on this page.

To check which card is on board, use the command :

```bash
lspci -k
```

In my case, the lines concerned are as follows:

```bash
Network controller: Broadcom Inc. and subsidiaries BCM4360 802.11ac Wireless Network Adapter (rev 03)
Subsystem: Apple Inc. BCM3460 802.11ac Wireless Network AdapterKernel driver in use: bcma-pci-bridge
Kernel modules: bcma, wl
```

If the `iw dev` command doesn't return anything, it is because the driver (`broadcom-wl` in our case), although present in the installer, is not loaded. We start by disabling the concurrent modules, which could be run :


```bash
rmmod b43 bcma ssb wl
```

Then, we activate:

```bash
modprobe wl
```

If all goes well, `iw dev` now displays something:

```bash
phy#1
    Interface wlan0
        ifindexwdev 0x100000001
        addr 00:00:00:00:00:00
        type managedtxpower 200.00 dBm
```

You can then activate the connection and then scan the surrounding networks (if the interface has, in the result of the previous command, a name other than `wlan0`, you have to adapt it) :

```bash
ifconfig wlan0 up
iwlist wlan0 scan
```

You will find other cases corresponding to other cards on the [MacbookPro11,x](https://wiki.archlinux.org/index.php/MacBookPro11,x) and [Broadcom](https://wiki.archlinux.org/index.php/Broadcom_wireless) in the wiki.

If you can't connect, you can still use a small USB wifi adapter to continue the installation, or use the "modem" mode of your Android phone (see [Android tethering](https://wiki.archlinux.org/index.php/Android_tethering) on the wiki).

To connect to the wifi, use the command (where `ssid` and `password` are respectively the name and password of your access point) :

```bash
wpa_supplicant -B -i wlan0 -c <(wpa_passphrase "ssid" "password")
```

Be careful not to put a space between the `<` and the `(`, to avoid getting the error `zsh: number expected`. Also, depending on the situation, it may not work on some channels in the 5 GHz band. In this case, the simplest way is to reconfigure your wifi on the 2.4 GHz band.

We get an IP address with the following command (which should display a succession of communications):

```bash
dhcpcd
```

We're checking that we're connected with :

```bash
ping archlinux.org
```

### Preparation of the installation

The following steps are common to all Arch Linux installations, and should be followed instead on the [installation guide](https://wiki.archlinux.org/index.php/installation_guide) on the wiki.

The internal clock is updated with :

```bash
timedatectl set-ntp true
```

The disks are partitioned, following the [corresponding] page (https://wiki.archlinux.org/index.php/Partitioning). A prior backup of the data is of course essential. Once this is done, you can check the existing disks and partitions with :

```bash
fdisk -l
```

We start by formatting the main partition (`/`), in this example `/dev/sda3` :

```bash
mkfs.ext4 /dev/sda3
```

Then we mount this partition so that it is accessible, during installation, in `/mnt` :

```bash
mount /dev/sda3 /mnt
```

We do the same with the boot partition (`/boot`), in our example located in `/dev/sda1`, in `/mnt/boot` :

```bash
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
```

### Installing Arch Linux

Before starting the installation, you can select a server near you to retrieve the data :

```bash
nano /etc/pacman.d/mirrorlist
```

Then we launch the basic installation on our main stake:

```bash
pacstrap /mnt base linux linux-firmware
```

We generate an `fstab' file:

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

We can then enter the newly created system:

```bash
arch-chroot /mnt
```

We start by configuring the time zone :

```bash
ln -sf /usr/share/zoneinfo/[Region]/[City] /etc/localtime
hwclock --systohc
```

We choose the languages that should be available in our system:

```bash
pacman -S nano
nano /etc/locale.gen
```

In my case, I needed to uncomment the lines `en_US.UTF-8` and `fr_FR.UTF-8`. We then generate the locales with :

```bash
locale-gen
```

The default language of the terminal is also selected with :

```bash
nano /etc/locale.conf
```

In my case, I use:

```bash
LANG=fr_FR.UTF-8
```

The keyboard configuration is also modified, to have the same as the one defined earlier, as well as the display font :

```bash
nano /etc/vconsole.conf
```

I use:

```bash
KEYMAP=fr-latin1
FONT=sun12x22
```

To prepare the network connection, let's give this computer a name:

```bash
nano /etc/hostname
```

For example:

```bash
myhostname
```

We then modify the following file :

```bash
nano /etc/hosts
```

We put:

```bash
127.0.0.1   localhost
::1         localhost
127.0.1.1   myhostname.localdomain    myhostname
```

Finally, choose a password for the administrator account with :

```bash
passwd
```


### Wifi driver

We still have to install our driver on our system to have a network connection on reboot. Let's start by retrieving the tools to be able to connect:

```bash
pacman -Syu iw wpa_supplicant dhcpcd
```

With a Broadcom BCM4360 (rev 03) card, we will install the `broadcom-wl` package in its [DKMS](https://en.wikipedia.org/wiki/Dynamic_Kernel_Module_Support) version. This will allow it to be upgraded along with the Linux kernels.

```bash
pacman -Syu linux-headers broadcom-wl-dkms
```

Finally, we generate:

```bash
mkinitcpio -p linux
```


### Bootloader

Everything is almost ready: we just have to install a bootloader so that our computer can launch Arch Linux at boot time.

With a UEFI system, the `systemd-boot' loader is simple to install (see [systemd-boot](https://wiki.archlinux.org/index.php/systemd-boot) on the wiki).

To do so:

```bash
bootctl --path=/boot install
```

For easy access to our score, we give it a label :

```bash
e2label /dev/sda3 "arch"
```

We then add an entry :

```bash
nano /boot/loader/entries/arch.conf
```

We put:

```bash
title   Arch
linux   /vmlinuz-linux
initrd  /initramfs-linux.img
options root=LABEL=arch rw
```

## First launch

You can exit the installer with `exit`, then `reboot.` On reboot, use the `alt` key again and select `Arch`.

Once the system is booted, you can use the user `root' and the password given earlier.


### Wifi connection

If everything has been installed correctly, we should get the card identifier (here `wlp3s0`) with :

```bash
iw dev
```

You can then log in with :

```bash
wpa_supplicant -B -i wlp3s0 -c <(wpa_passphrase "ssid" "password")
dhcpcd
```

If you get the message `no interface have a carrier`, you can try running `dhcpcd --release` and then `dhcpcd` again.

### Creating a user

In order not to remain on the administrator account, we create a user account and give him a password with :

```bash
useradd -m your_username
passwd your_username
```

To give him `sudo' access, we install :

```bash
pacman -Syu sudo
```

Let's edit:

```bash
nano /etc/sudoers
```

We add:

```bash
your_username ALL=(ALL) ALL
```

### What's next?

It's up to you! The page [General recommendations](https://wiki.archlinux.org/index.php/General_recommendations) details the different steps you may wish to perform (graphics system, drivers, multimedia, network).

In addition, the page [MacbookPro 11,x](https://wiki.archlinux.org/index.php/MacBookPro11,x) details the various subjects specific to the computer (graphic card if you have one, fan management, webcam).

Finally, managing the HiDPI (Retina) screen may require a lot of testing, depending on your configuration. The [HiDPI](https://wiki.archlinux.org/index.php/HiDPI) page of the wiki explores the problem thoroughly.
