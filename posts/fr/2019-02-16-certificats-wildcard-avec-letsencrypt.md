---
title: Certificats <em>wildcard</em><br/> avec Let’s Encrypt
original: 2018-03-17
---

Lancé fin 2015, l'organisme à but non lucratif *[Let’s Encrypt](https://letsencrypt.org)* a permis de démocratiser l'utilisation de HTTPS en fournissant gratuitement des certificats SSL grâce à un système de validation automatisé.

Après plusieurs mois de tests, *Let’s Encrypt* vient de lancer la deuxième version de son client ([ACME v2](https://community.letsencrypt.org/t/acme-v2-production-environment-wildcards/55578)) avec une fonctionnalité très demandée : pouvoir obtenir des certificats *wildcard*, valables pour tous les sous-domaines d'un même domaine.

Cette possibilité est particulièrement intéressante lorsque l'on utilise de nombreux sous-domaines : jusqu'ici, il était nécessaire de rééditer un nouveau certificat pour ajouter un nouveau sous-domaine, ou pour en supprimer un ancien. Désormais, un simple `*.domain.tld` suffit !


### Installation

Pour obtenir nos certificats, nous allons utiliser le client `certbot`. Le site propose plusieurs [méthodes d'installation](https://certbot.eff.org) selon votre plateforme. Depuis Debian, on utilise simplement :


```none
sudo apt-get install letsencrypt

```

### Demande d'un certificat

Attention, si vous souhaitez un certificat à la fois pour la racine du domaine (`domain.tld`) et l'ensemble de ses sous-domaines (`*.domain.tld`), les deux doivent être spécifiés. Avec l'option `-d`, il est possible de lister les domaines et sous-domaines souhaités :

```none
sudo letsencrypt certonly --manual --preferred-challenges dns --register -d domain.tld -d *.domain.tld
```

### Validation

*Let’s encrypt* va désormais devoir nous demander de prouver que l'on possède bien le contrôle sur les noms de domaine demandés. Il doit vérifier que vous possédez l'intégralité du nom de domaine, et demande pour cela la création d'un enregistrement TXT spécifique dans la zone DNS du nom de domaine, ce qui est réalisable depuis votre registar :

```
-------------------------------------------------------------------------------
Please deploy a DNS TXT record under the name
_acme-challenge.domain.tld with the following value:

c81US66r6JVk1LwyFHbzINQvIU_m5gJWXgcUm8Qj2

Before continuing, verify the record is deployed.
-------------------------------------------------------------------------------
Press Enter to Continue
```

Deux enregistrements TXT peuvent être demandés, c'est parfaitement normal (à la fois pour `domain.tld` et `*.domain.tld`).

Une fois créé, le certificat se situe dans `/etc/letsencrypt/live/domain.tld`.


### Renouvellement des certificats

Pour renouveler régulièrement le certificat, il suffit de créer une tâche cron.

```sudo crontab -e```

Par exemple, pour renouveler le certificat tous les lundis dans la nuit, et redémarrer nginx dans la foulée, on indique :

```
30 4 * * 1 letsencrypt renew
40 4 * * 1 /etc/init.d/nginx reload
```
