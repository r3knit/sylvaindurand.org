---
title: Send emails with msmtp
alt: Send emails <br> with *msmtp*
aliases: send-emails-with-ssmtp
categories: Linux
---

There are many situations in which sending emails from a server can be useful, either because applications require it, or to do *monitoring*, such as being notified of security updates available on your system or the proper execution of *cron* tasks.

However, configuring a mail server with `postfix`, `exim` or `sendmail` requires a large number of steps to be performed and a careful configuration, due to all the devices set up to fight spam{{% marginalia %}}including DKIM, SPF, DMARC, static IP and Reverse DNS, white list...{{% /marginalia %}}, which makes the task very tedious for a simple personal server.

Nevertheless, it is possible to simply use an existing email account, and send *via* SMTP emails as a traditional email client would.

Until now, I used to use *ssmtp* for this, but this one is no longer maintained, and it is no longer possible to install it from Debian 10 *Buster*. We will use *msmtp*, which is just as easy to use and efficient.



### Installation of msmtp

From a *Debian* based system, you can simply install the following packages :

```
sudo apt-get install msmtp msmtp-mta
```

The configuration file is as follows:

```
sudo nano /etc/msmtprc
```

Then use the following settings with the IDs and settings of your email provider:

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

Or, with OVH :

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

Once the file has been saved, we can try to send our first emails.


### Test sending emails

It is possible to use `msmtp` directly to send your email:

```bash
echo "Message" | mail -s "Title" <email-adress>
```


### Change the sender's name

It is possible to define, for each user of your server, a personalized sender name when sending emails:

```bash
sudo chfn -f 'Custom name for root' root
sudo chfn -f 'Custom name for user' <user>
```

### Example of use: apticron

If you have a server, you should always be informed of the latest updates available for your system.

If you use a system based on *Debian*, and you use `apt` to install or update your packages, the little `apticron` utility automatically sends you an email as soon as an update is available.

It is installed with:

```
sudo apt-get install apticron
```

To indicate your email address, simply change it:

```
sudo nano /etc/apticron/apticron.conf
```

Finally, indicate the address at which you wish to receive the notifications:

```
EMAIL="<your-email>"
```