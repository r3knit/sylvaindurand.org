---
title: Arch Linux sur Macbook Pro fin 2013
alt: <em>Arch Linux</em> sur <br>Macbook Pro fin 2013
categories: Linux
---

Arch Linux est une distribution dont la procédure d'installation peut sembler exigente, mais qui offre en contrepartie un contrôle complet de ce qui est installé, mais surtout permet d'expérimenter et de comprendre beaucoup d'éléments de base du fonctionnement d'un système Linux.

Cet article détaille les étapes qui me permettent d'obtenir une installation minimale fonctionnelle sur un MacBook Pro 13" de fin 2013 (`Macbook Pro 11,1`). Celui-ci se caractérise par quelques spécificités, notamment une carte wifi *Broadcom* récalcitrante.

Il est *rigoureusement déconseillé de suivre ces commandes aveuglément*, car Arch Linux est une distribution qui est très régulièrement mise à jour et que chaque système est différent.

Commencez par ouvrir et suivre scrupuleusement le [guide d'installation](https://wiki.archlinux.org/index.php/installation_guide) et la page [MacbookPro11,x](https://wiki.archlinux.org/index.php/MacBookPro11,x) du [wiki officiel](https://wiki.archlinux.org/), particulièrement complets. Cet article pourra servir  de complément à ces deux pages.


## Préparation du support d'installation


Commençons par télécharger l'image d'installation d'Arch Linux depuis le [site officiel](https://www.archlinux.org/download/). Nous utiliserons ici une clef USB pour procéder à l'installation. Attention : tout son contenu sera supprimé !


### Depuis macOS

Pour connaître le nom de la clef, on utilise :

```bash
diskutil list
```

On démonte alors la clef, puis on procède à la copie de l'image (où `diskx` est à modifier par le nom de votre clef) :

```bash
sudo diskutil unmountdisk /dev/diskx
sudo dd bs=1m if=archlinux_image.iso of=/dev/rdiskx conv=sync
```

### Depuis Linux

Pour connaître le nom de notre clef USB, nous pouvons utiliser :

```bash
fdisk -l
```

Pour copier l'image sur la clef, on peut utiliser la commande suivante (où `sdx` est à remplacer par le nom de votre clef). On redémarre en appuyant sur alt.

```bash
dd bs=4M if=archlinux_image.iso of=/dev/sdx status=progress oflag=sync
```

## Installation

Nous pouvons désormais redémarrer l'ordinateur depuis la clef USB, en appuyant sur la touche Alt au moment de l'initialisation de l'ordinateur (après le son d'initialisation de l'ordinateur). On choisir le menu `EFI Boot`, puis `Arch Linux archiso x86_UEFI CD`. Nous arrivons alors sur l'installeur.

### Disposition du clavier

Par défaut, le clavier est en Qwerty. Dans mon cas, pour obtenir un clavier Azerty, on utilise :

```bash
loadkeys fr-latin1
```

Pour le taper, le `A` est à la place du `Q`, le `-` à la place du `°`, et le `1` se tape sans la touche majuscule. Par ailleurs, cela nous donne un clavier de type Windows, différent de la variante utilisée sur les ordinateurs Apple. En particulier, le `-` sur trouve sur la touche `6` et le `_` sur la touche `8`.

### Affichage

Du fait de l'écran en HiDPI (Retina), les écritures sont très petites. Il nous est possible, pour ce terminal, d'obtenir une police plus grande avec :

```bash
setfont sun12x22
```

### Wifi

Il s'agit de la partie la plus bloquante par rapport aux ordinateurs habituels. Le MacBook Pro de fin 2013 embarque le plus souvent une carte wifi *Broadcom*, connue pour être parfois difficile à utiliser avec les pilotes habituels.

L'objet de cette partie est uniquement de disposer d'une connexion réseau pour faire l'installation, nous ne configurons par encore le wifi pour notre système final. Si votre système est déjà installé et que vous cherchez à avoir un wifi fonctionnel, vous pouvez aller voir plus bas sur cette page.

Pour vérifier la carte qui est embarquée, on utilise la commande :

```bash
lspci -k
```

Dans mon cas, les lignes concernées sont les suivantes :

```bash
Network controller: Broadcom Inc. and subsidiaries BCM4360 802.11ac Wireless Network Adapter (rev 03)
Subsystem: Apple Inc. BCM3460 802.11ac Wireless Network AdapterKernel driver in use: bcma-pci-bridge
Kernel modules: bcma, wl
```

Si la commande `iw dev` ne retourne rien, c'est parce que le pilote (`broadcom-wl` dans notre cas), bien que présent dans l'installeur, n'est pas chargé. On commence par désactiver les modules concurrents, qui pourraient être lancés :

```bash
rmmod b43 bcma ssb wl
```

Puis on active :

```bash
modprobe wl
```

Si tout va bien, `iw dev` affiche désormais quelque chose :

```bash
phy#1
    Interface wlan0
        ifindexwdev 0x100000001
        addr 00:00:00:00:00:00
        type managedtxpower 200.00 dBm
```

On peut alors activer la connexion puis scanner les réseaux environnants (si l'interface a, dans le résultat de la commande précédente, un autre nom que `wlan0`, il faut l'adapter) :

```bash
ifconfig wlan0 up
iwlist wlan0 scan
```

Vous trouverez d'autres cas correspondant à d'autres cartes sur les pages [MacbookPro11,x](https://wiki.archlinux.org/index.php/MacBookPro11,x) et [Broadcom](https://wiki.archlinux.org/index.php/Broadcom_wireless) du wiki.

Si vous ne parvenez pas à vous connecter, il reste possible, pour poursuivre l'installation, d'utiliser un petit adaptateur wifi par USB, ou d'utiliser le mode "modem" de votre téléphone Android (voir [Android tethering](https://wiki.archlinux.org/index.php/Android_tethering) sur le wiki).


Pour se connecter au wifi, on utilise alors la commande (où `ssid` et `password` sont respectivement le nom et le mot de passe de votre point d'accès) :

```bash
wpa_supplicant -B -i wlan0 -c <(wpa_passphrase "ssid" "password")
```

Attention, il ne faut pas mettre d'espace entre le `<` et le `(`, pour éviter d'avoir l'erreur `zsh: number expected`. Par ailleurs, selon les situations, il est possible que cela ne fonctionne pas sur certains canaux de la bande des 5 GHz. Dans ce cas, le plus simple est de reconfigurer son wifi sur la bande des 2,4 GHz.

Après quelques secondes d'attente, on récupère enfin une adresse IP avec la commande suivante (qui doit afficher une succession de communications) :

```bash
dhcpcd
```

On vérifie que l'on est connecté avec :

```bash
ping archlinux.org
```

### Préparation de l'installation

La suite des étapes est commune à toutes les installations d'Arch Linux, et devrait être suivie plutôt sur le [guide d'installation](https://wiki.archlinux.org/index.php/installation_guide) du wiki.

On met à jour l'horloge interne avec :

```bash
timedatectl set-ntp true
```

On partionne les disques, en suivant la page [correspondante](https://wiki.archlinux.org/index.php/Partitioning). Une sauvegarde préalable des données est bien sûr indispensable. Une fois cela fait, on peut vérifier les disques et partitions existantes avec :

```bash
fdisk -l
```

On commence par formater la partition principale (`/`), dans cet exemple `/dev/sda3` :

```bash
mkfs.ext4 /dev/sda3
```

Puis on monte cette partition pour qu'elle soit accessible, au cours de l'installation, dans `/mnt` :

```bash
mount /dev/sda3 /mnt
```

On fait de même avec la partition de démarrage (`/boot`), dans notre exemple située dans `/dev/sda1`, dans `/mnt/boot` :

```bash
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
```

### Installation d'Arch Linux

Avant de lancer l'installation, vous pouvez sélectionner un serveur proche de chez vous pour récupérer les données :

```bash
nano /etc/pacman.d/mirrorlist
```

Puis on lance l'installation de base sur notre participation principale :

```bash
pacstrap /mnt base linux linux-firmware
```

On génère un fichier `fstab` :

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

Nous pouvons alors rentrer dans le système nouvellement créé :

```bash
arch-chroot /mnt
```

On commence par configurer le fuseau horaire :

```bash
ln -sf /usr/share/zoneinfo/[Region]/[City] /etc/localtime
hwclock --systohc
```

On choisit les langues qui doivent être disponibles dans notre système :

```bash
pacman -S nano
nano /etc/locale.gen
```

Dans mon cas, il s'agit de décommenter les lignes `en_US.UTF-8` et `fr_FR.UTF-8`.
On génère alors les *locales*  avec :

```bash
locale-gen
```

On choisit également la langue par défaut du terminal avec :

```bash
nano /etc/locale.conf
```

Dans mon cas, j'utilise :

```bash
LANG=fr_FR.UTF-8
```

On modifie également la configuration du clavier, pour avoir la même que celle définie plus tôt, ainsi que la police d'affichage :

```bash
nano /etc/vconsole.conf
```

J'utilise :

```bash
KEYMAP=fr-latin1
FONT=sun12x22
```

Pour préparer la connexion réseau, donnons un nom à cet ordinateur :

```bash
nano /etc/hostname
```

Par exemple :

```bash
myhostname
```

On modifie ensuite le fichier suivant :

```bash
nano /etc/hosts
```

On y indique :

```bash
127.0.0.1   localhost
::1         localhost
127.0.1.1   myhostname.localdomain    myhostname
```

Enfin, on choisit un mot de passe pour le compte administrateur avec :

```bash
passwd
```


### Pilote Wifi

Il nous reste à installer notre pilote sur notre système pour avoir une connexion réseau au redémarrage. Commençons par récupérer les outils pour pouvoir nous connecter :

```bash
pacman -Syu iw wpa_supplicant dhcpcd
```

Avec une carte Broadcom BCM4360 (rev 03), on va installer le paquet `broadcom-wl` dans sa version [DKMS](https://en.wikipedia.org/wiki/Dynamic_Kernel_Module_Support). Celle-ci lui permettra d'être mise à jour en même temps que les noyaux Linux.

```bash
pacman -Syu linux-headers broadcom-wl-dkms
```


Enfin, on génère :

```bash
mkinitcpio -p linux
```


### Chargeur d'amorce

Tout est presque prêt : il ne reste plus qu'à installer un chargeur d'amorce (*"bootloader"*) pour que notre ordinateur puisse lancer Arch Linux au démarrage.

S'agissant d'un système UEFI, le chargeur `systemd-boot` est simple à installer (voir [systemd-boot](https://wiki.archlinux.org/index.php/systemd-boot) sur le wiki).

On lance pour cela :

```bash
bootctl --path=/boot install
```

Pour accéder facilement à notre partition, on lui donne un label :

```bash
e2label /dev/sda3 "arch"
```

On ajoute alors une entrée :

```bash
nano /boot/loader/entries/arch.conf
```

On y met :

```bash
title   Arch
linux   /vmlinuz-linux
initrd  /initramfs-linux.img
options root=LABEL=arch rw
```

## Premier lancement

On peut quitter l'installeur avec `exit`, puis `reboot.` Au redémarrage, on utilise à nouveau la  touche `alt` et on sélectionne `Arch`.

Une fois le système démarré, on peut utiliser l'utilisateur `root` et le mot de passe indiqué plus tôt.


### Connexion au Wifi

Si tout a bien été installé, on devrait obtenir l'identifiant de la carte (ici `wlp3s0`) avec :

```bash
iw dev
```

On peut alors se connecter avec :

```bash
wpa_supplicant -B -i wlp3s0 -c <(wpa_passphrase "ssid" "password")
dhcpcd
```

Si vous recevez le message `no interface have a carrier`, vous pouvez essayer de lancer `dhcpcd --release` puis, à nouveau, `dhcpcd`.

### Création d'un utilisateur

Pour ne pas rester sur le compte administrateur, on créé un compte utilisateur et on lui donne un mot de passe avec :

```bash
useradd -m your_username
passwd your_username
```

Pour lui donner un accès `sudo`, on installe :

```bash
pacman -Syu sudo
```

On modifie :

```bash
nano /etc/sudoers
```

On y ajoute :

```bash
your_username ALL=(ALL) ALL
```

### Et ensuite ?

Cela dépend de vous ! La page *[General recommandations](https://wiki.archlinux.org/index.php/General_recommendations)* détaille les différentes étapes que vous pourriez souhaiter réaliser (système graphique, pilotes, multimédia, réseau).

Par ailleurs, la page [MacbookPro 11,x](https://wiki.archlinux.org/index.php/MacBookPro11,x) détaille les différents sujets spécifiques à l'ordinateur (carte graphique si vous en avez une, gestion des ventilateurs, webcam).

Enfin, la gestion de l'écran HiDPI (Retina) pourra vous demander pas mal d'essais, selon votre configuration. La page [HiDPI](https://wiki.archlinux.org/index.php/HiDPI) du wiki explore de façon complète le problème.
