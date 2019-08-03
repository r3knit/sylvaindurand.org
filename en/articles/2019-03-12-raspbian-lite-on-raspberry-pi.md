---
title: Raspbian Lite on Raspberry Pi
alt: <em>Raspbian Lite</em> <br> on Raspberry Pi
categories: Raspberry Pi
---


More than four years ago, I wrote an article explaining how to install [ArchLinux on Raspberry Pi](/installing-archlinux-on-raspberry-pi/). I have since migrated to the [Raspbian](https://www.raspberrypi.org/downloads/raspbian/) distribution, based on [Debian](https://www.debian.org) and made specifically for the Raspberry Pi.

Historically very large, because it has a graphic interface and several specific software, *Raspbian* now exists in a *Light* version that allows you to limit the packages to the essentials, to manage your Raspberry Pi from the command line.

This article is a small personal memo on the steps to install and secure *Raspbian* to get a Raspberry Pi ready to use, usable *via* command line with SSH.


{{% large %}}
![Raspberry Pi – Jonathan Rutheiser, CC BY-SA 3.0](/img/raspberry/raspberrypi.svg)
{{% /large %}}

## Downloading

First we download the image of *Raspbian* from the [following address](https://www.raspberrypi.org/downloads/raspbian/). Select "Raspbian Stretch Lite". The official site offers user manuals for [Linux](https://www.raspberrypi.org/documentation/installation/installing-images/linux.md), [MacOS](https://www.raspberrypi.org/documentation/installation/installing-images/mac.md) and [Windows](https://www.raspberrypi.org/documentation/installation/installing-images/windows.md).

On MacOS, we start by looking at the address of the SD card:

```
diskutil list
```

Once you know the SD card number, which appears as `/dev/disk<number>`, you unmount the volume, then write the image on it :

```
diskutil unmountDisk /dev/disk<number>
sudo dd bs=1m if=image.img of=/dev/rdisk<number> conv=sync
```

To be able to connect with SSH, we must create a file named `ssh` at the root of our SD card:

```
touch /Volumes/boot/ssh
```

If you need to connect on a Wifi network at boot:

```
nano /Volumes/boot/wpa_supplicant.conf
```

We put:

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
network={
    ssid="<your-network-ssid>"
    psk="<your-network-password>"
    key_mgmt=WPA-PSK
}
```



Finally, we eject the card:

```
sudo diskutil eject /dev/rdisk<number>
```


## First connection

Once the Raspberry is powered up with the SD card, you can connect with SSH:

```
ssh pi@<raspberry-pi-ip>
```

The default password is `raspberry`. Let's start by changing it:

```
passwd
```

### Updating

After setting up a new server, we start by updating all the packages:

```
sudo apt-get update && sudo apt-get upgrade
sudo apt-get dist-upgrade
sudo apt autoremove
```


### Language settings

In order to set the language and time zone, launch:

```
sudo dpkg-reconfigure locales
sudo dpkg-reconfigure tzdata
```

### Security

We will prevent the `root' connection and change ports, in order to prevent a large number of attack attempts. To do this, we edit:

```
sudo nano /etc/ssh/sshd_config
```

We modify the following parameters with:

```
Port <your-custom-port>
LoginGraceTime 2m
PermitRootLogin no
StrictModes yes
PermitEmptyPasswords no
```

We finally restart the Raspberry Pi:

```
sudo reboot
```


### Connection via SSH

Locally, we create a new private key:

```
ssh-keygen -t ed25519
````

Then we edit `.ssh/config` to connect easily:

```
Host pi
Hostname <raspberry-pi-ip>
Port <your-custom-port>
User pi
IdentityFile ~/.ssh/<your-private-key>
```


Then we send the public key to the server:

```
ssh-copy-id -i ~/.ssh/<your-private-key> pi
```

You can then connect directly with `ssh vps`.

The system is now fully operational!
