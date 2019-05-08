---
title: Installer Pi-hole avec PiVPN
alt: Installer *Pi-hole* <br> avec *PiVPN*
categories: Raspberry Pi
---

Si les bloqueurs de publicité restent très efficaces au sein des navigateurs, publicités et traqueurs restent largement indemnes en dehors, qu'il s'agisse des ordinateurs, téléphones, télévisions et autres objets connectés.

[*Pi-hole*](https://pi-hole.net) est une application qui permet de bloquer les publicités et les traqueurs en agissant directement au niveau de la résolution des DNS. De façon générale, il donne une information relativement exhaustive d'un grand nombre de connexions qui passent sur votre réseau.

Combiné avec [*PiVPN*](http://www.pivpn.io), il devient possible d'utiliser *Pi-hole* en itinérance, depuis votre téléphone ou sur un réseau public, tout en chiffrant l'intégralité de vos connexions.

{{% large %}}
[![PiHole](/img/raspberry/pihole.png)](/img/raspberry/pihole.png)
{{% /large %}}

## Installation de Pi-hole

Cet article suppose que vous avez un Raspberry Pi prêt à l'emploi ! Si ce n'est pas le cas, vous pouvez jeter un coup d'œil sur comment [installer Raspbian](/raspbian-lite-sur-raspberry-pi/) ou [Archlinux](/installer-archlinux-sur-raspberry-pi/) sur votre Raspberry Pi.

Il est par ailleurs important que votre routeur attribue une adresse IP fixe au Raspberry Pi, afin de pouvoir le déclarer celui-ci comme hôte DNS depuis vos appareils.

L'installation de *Pi-hole* se fait simplement à l'aide d'une seule commande :

```
curl -sSL https://install.pi-hole.net | bash
```

À titre personnel, j'ai choisi les serveurs DNS de [Quad9](https://www.quad9.net) (`9.9.9.9`).

Une fois l'installation terminée, on peut changer le mot de passe :

```
sudo pihole -a -p
```

Trois possibilités s'offrent alors à vous pour vous assurer que votre ordinateur utilise bien votre Raspberry Pi pour résoudre les DNS.

Le plus simple est d'activer le serveur DHCP de Pi-Hole : depuis le serveur web, on va dans `Settings`, puis dans `DHCP`, avant de sélectionner `DHCP server enabled` ; il faut ensuite se rendre dans l'interface d'administration de votre routeur ou de votre box pour désactiver le DHCP.

Une autre solution est, dans l'interface de paramétrage de votre routeur ou votre box internet, de déclarer l'IP de votre Raspberry Pi comme champ de serveur DNS.

Si aucune de ces solutions n'est possible, il est nécessaire de déclarer votre Raspberry Pi comme serveur DNS dans les paramètres réseaux de l'ensemble de vos périphériques.

Redémarrez votre connexion au réseau : l'interface d'administration de *Pi-hole* devrait désormais être disponible depuis `http://pi.hole`.

Vous pouvez alors y déclarer les listes de blocage pour commencer à bloquer des publicités. À titre personnel, j'utilise les sources de [wally3k](https://wally3k.github.io/) et de [tspprs](https://tspprs.com/). Vous pouvez également bloquer des sites particuliers !


## Installation de PiVPN

L'installation se fait là encore avec une seule ligne de commande :

```
curl -L https://install.pivpn.io | bash
```

Lorsqu'il est demandé quel serveur DNS choisir, on choisit `Custom`, puis l'on indique l'IP du Raspberry Pi.

Une fois l'installation effectuée, retournons dans l'interface d'administration de *Pi-hole*, pour aller dans `Settings`, `DNS`, et dans `Interface listening behavior` : on utilise `Listen on all interfaces` pour s'assurer que *PiVPN* communique bien avec *Pi-hole*.

On peut simplement créer un nouveau profil VPN avec :

```
pivpn add
```

Pour récupérer le profil, on utilise en local :

```
scp pi:/home/pi/ovpns/<your-profile>.ovpn .
```

Il est alors désormais possible de se connecter, depuis un client OpenVPN, sur un ordinateur ou un téléphone, pour bénéficier du filtrage de de *Pi-hole*.

Assurez-vous que votre client ne dispose pas d'une option permettant d'utiliser d'autres DNS (le client pour iOS, par exemple, se rabat automatiquement vers les serveurs DNS de Google).
