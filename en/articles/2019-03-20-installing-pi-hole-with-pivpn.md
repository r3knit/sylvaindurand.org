---
title: Installing Pi-hole with PiVPN
alt: Installing *Pi-hole* <br> with *PiVPN*
categories: Raspberry Pi
---

While ad blockers remain highly effective within browsers, advertisements and trackers remain largely intact outside, whether it is computers, phones, televisions and other connected objects.

[*Pi-hole*](https://pi-hole.net) is an application that allows you to block ads and trackers by acting directly at the DNS resolution level. In general, it provides relatively comprehensive information on a large number of connections that pass through your network.

Combined with [*PiVPN*](http://www.pivpn.io), it becomes possible to use *Pi-hole* when roaming, from your phone or on a public network, while encrypting all your connections.

{{% large %}}
[![PiHole](/img/raspberry/pihole.png)](//img/raspberry/pihole.png)
{{% /large %}}

## Installation of Pi-hole

This article assumes that you have a Raspberry Pi ready to use! If not, you can take a look at how to [install Raspbian](/raspbian-lite-on-raspberry-pi/) ou [Archlinux](/installing-archlinux-on-raspberry-pi/) on your Raspberry Pi.

It is also important that your router assigns a fixed IP address to the Raspberry Pi, so that it can be declared as a DNS host from your devices.

The installation of *Pi-hole* is done simply with a single command:

```
curl -sSL https://install.pi-hole.net | bash
```

On a personal note, I chose [Quad9](https://www.quad9.net)'s DNS servers (`9.9.9.9`).

Once the installation is complete, you can change the password:

```
sudo pihole -a -p
```

There are three ways to ensure that your computer is using your Raspberry Pi to resolve DNS.

The easiest way is to activate the Pi-Hole DHCP server: from the web server, you go to `Settings`, then `DHCP`, before selecting `DHCP server enabled`; then you have to go to the administration interface of your router or your box to disable DHCP.

Another solution is, in the configuration interface of your router or internet box, to declare the IP of your Raspberry Pi as a DNS server field.

If none of these solutions are possible, it is necessary to declare your Raspberry Pi as a DNS server in the network settings of all your devices.


Redémarrez votre connexion au réseau : l'interface d'administration de *Pi-hole* devrait désormais être disponible depuis `http://pi.hole/`.

VYou can then declare the blocking lists there to start blocking ads. As an individual, I use the sources of [wally3k](https://wally3k.github.io/) and [tspprs](https://tspprs.com/). You can also block particular sites!


## Installation of PiVPN

The installation is done again with a single command line:

```
curl -L https://install.pivpn.io | bash
```

When asked which DNS server to choose, select `Custom`, then indicate the IP of the Raspberry Pi.

Once the installation is done, let's go back to the administration interface of *Pi-hole*, to go to `Settings`, `DNS`, and `Interface listening behavior`: we use `Listen on all interfaces` to make sure that *PiVPN* is communicating with *Pi-hole*.

You can simply create a new VPN profile with:

```
pivpn add
```

To retrieve the profile, we use locally:

```
scp pi:/home/pi/ovpns/<your-profile>.ovpn .
```

It is now possible to connect, from an OpenVPN client, to a computer or a phone, to benefit from the filtering of *Pi-hole*.

Make sure your client does not have an option to use other DNS (the client for iOS, for example, automatically switches to Google's DNS servers).
