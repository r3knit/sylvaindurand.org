---
title: Envoyer des mails avec msmtp
alt: Envoyer des mails <br> avec *msmtp*
aliases: send-emails-with-ssmtp
categories: Linux
---

Il existe plein de situations dans lesquelles envoyer des emails depuis un serveur peut-être utile, soit parce que des applications le nécessitent, soit pour faire du *monitoring*, comme être averti de mises à jour de sécurité disponibles sur son système ou de la bonne exécution de tâches *cron*.

Pour autant, configurer un serveur mail avec `postfix`, `exim` ou `sendmail` nécessite un grand nombre d'étapes à réaliser et une configuration minutieuse, du fait de tous les dispositifs mis en place pour lutter contre le spam{{% marginalia %}}notamment DKIM, SPF, DMARC, IP statique et Reverse DNS, inscription sur listes blanches...{{% /marginalia %}}, ce qui rend la tâche très fastidieuse pour un simple serveur personnel.

Néanmoins, il est possible de tout simplement utiliser un compte email déjà existant, et d'envoyer les mails *via* SMTP comme le ferait un client mail traditionnel. 

Jusqu'ici, j'utilisais pour cela *ssmtp*, mais celui-ci n'est plus maintenu, et il n'est plus possible de l'installer depuis Debian 10 *Buster*. Nous allons utiliser *msmtp*, tout aussi simple d'utilisation et efficace.


### Installation de msmtp

Depuis un système basé sur *Debian*, on peut simplement installer les paquets suivants :

```
sudo apt-get install msmtp msmtp-mta
```

Le fichier de configuration est le suivant :

```
sudo nano /etc/msmtprc
```

Utilisez alors les paramètres suivants avec les identifiants et paramètres de votre fournisseur d'emails :

```bash
defaults
auth           on
tls            on
tls_trust_file /etc/ssl/certs/ca-certificates.crt
logfile        ~/.msmtp.log
account        gmail
host           smtp.gmail.com
port           587
from           username@gmail.com
user           username
password       password
account default : gmail
```

Par exemple, avec OVH :

```bash
defaults
auth           on
tls            on
tls_trust_file /etc/ssl/certs/ca-certificates.crt
logfile        ~/.msmtp.log
account        ovh
host           ssl0.ovh.net
port           587
from           username@domain.tld
user           username@domain.tld
password       password
account default : ovh
```

Une fois le fichier enregistré, nous pouvons tenter d'envoyer nos premiers mails.


### Tester l'envoi d'emails

Il est possible d'utiliser directement `msmtp` pour envoyer votre email :

```bash
echo "Message" | mail -s "Title" <email-adress>
```


### Changer le nom de l'expéditeur

Il est possible de définir, pour chaque utilisateur de votre serveur, un nom d'expéditeur personnalisé lors de l'envoi des mail :

```bash
sudo chfn -f 'Custom name for root' root
sudo chfn -f 'Custom name for user' <user>
```

### Exemple d'utilisation : apticron

Si vous avez un serveur, vous devez être toujours averti des dernières mises à jour disponibles pour votre système.

Si vous utilisez un système basé sur *Debian*, et que vous utilisez `apt` pour installer ou mettre à jour vos paquets, le petit utilitaire `apticron` vous envoie automatiquement un mail dès qu'une mise à jour est disponible.

On l'installe avec :

```
sudo apt-get install apticron
```

Pour indiquer son adresse email, on modifie simplement :

```
sudo nano /etc/apticron/apticron.conf
```

Indiquez enfin l'adresse à laquelle vous souhaitez recevoir les notifications :

```
EMAIL="<your-email>"
```