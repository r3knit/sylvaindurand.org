---
title: Raspbian Lite sur Raspberry Pi
alt: <em>Raspbian Lite</em> <br> sur Raspberry Pi
aliases: installer-archlinux-sur-raspberry-pi
categories: Raspberry Pi
---


Il y a plus de quatre ans, j'ai rédigé un article expliquant comment installer [ArchLinux sur Raspberry Pi](/installer-archlinux-sur-raspberry-pi/). J'ai depuis migré vers la distribution [Raspbian](https://www.raspberrypi.org/downloads/raspbian/), basée sur [Debian](https://www.debian.org) et réalisée spécifiquement pour le Raspberry Pi.

Historiquement très volumineuse, car embarquant une interface graphique et plusieurs logiciels spécifiques, *Raspbian* existe désormais dans une version *Light* qui permet de se limiter au strict essentiel, pour gérer son Raspberry Pi en ligne de commandes.

Cet article constitue un petit mémo personnel sur les étapes permettant d'installer et de sécuriser *Raspbian* pour avoir un Raspberry Pi prêt à l'emploi, utilisable *via* ligne de commande avec SSH.

{{% large %}}
![Raspberry Pi – Jonathan Rutheiser, CC BY-SA 3.0](/img/raspberry/raspberrypi.svg)
{{% /large %}}

## Téléchargement

On commence par télécharger l'image de *Raspbian* depuis l'[adresse suivante](https://www.raspberrypi.org/downloads/raspbian/). Sélectionnez "Raspbian Stretch Lite". Le site officiel propose des notices d'utilisation pour [Linux](https://www.raspberrypi.org/documentation/installation/installing-images/linux.md), [MacOS](https://www.raspberrypi.org/documentation/installation/installing-images/mac.md) et [Windows](https://www.raspberrypi.org/documentation/installation/installing-images/windows.md).

Sur MacOS, on commence par regarder l'adresse de la carte SD :

```
diskutil list
```

Une fois que l'on connaît le numéro de la carte SD, qui apparaît sous la forme `/dev/disk<number>`, on démonte le volume, puis on écrit l'image dessus :

```
diskutil unmountDisk /dev/disk<number>
sudo dd bs=1m if=image.img of=/dev/rdisk<number> conv=sync
```

Pour pouvoir nous connecter avec SSH, on doit créer un fichier nommé `ssh` à la racine de notre carte SD :

```
touch /Volumes/boot/ssh
```

S'il y a besoin de se connecter à un réseau wifi au démarrage :

```
nano /Volumes/boot/wpa_supplicant.conf
```

On met :

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
network={
    ssid="<your-network-ssid>"
    psk="<your-network-password>"
    key_mgmt=WPA-PSK
}
```

Enfin, on éjecte la carte :

```
sudo diskutil eject /dev/rdisk<number>
```


## Première connexion

Une fois le Raspberry sous tension avec la carte SD, on peut se connecter avec SSH :

```
ssh pi@<raspberry-pi-ip>
```

Le mot de passe par défaut est `raspberry`. Commençons par le changer :
```
passwd
```

### Mise à jour

Après avoir mis en place un nouveau serveur, on commence par mettre à jour l'ensemble des paquets :

```
sudo apt-get update && sudo apt-get upgrade
sudo apt-get dist-upgrade
sudo apt autoremove
```


### Réglage de la langue

Afin de régler la langue et le fuseau horaire, on lance :

```
sudo dpkg-reconfigure locales
sudo dpkg-reconfigure tzdata
```

### Sécurité

Nous allons empêcher la connexion en `root` et changer de port, afin de se prémunir d'un grand nombre de tentatives d'attaque. Pour cela, on édite :

```
sudo nano /etc/ssh/sshd_config
```

On modifie les paramètres suivants en :

```
Port <your-custom-port>
LoginGraceTime 2m
PermitRootLogin no
StrictModes yes
PermitEmptyPasswords no
```

On redémarre enfin le Raspberry Pi :

```
sudo reboot
```


### Connexion via SSH

En local, on crée un nouvelle clef privée :

```
ssh-keygen -t ed25519 -a 100
```

Puis on édite `.ssh/config` pour se connecter facilement :

```
Host pi
    Hostname <raspberry-pi-ip>
    Port <your-custom-port>
    User pi
    IdentityFile ~/.ssh/<your-private-key>
```


Puis on envoie la clef publique sur le serveur :

```
ssh-copy-id -i ~/.ssh/<your-private-key> pi
```

On peut alors se connecter directement avec `ssh vps`.

Le système est désormais complètement opérationnel !
