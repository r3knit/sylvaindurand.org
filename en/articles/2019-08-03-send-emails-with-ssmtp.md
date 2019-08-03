---
title: Send emails with ssmtp
alt: Send emails <br> with <em>ssmtp</em>
categories: Linux
---

There are many situations in which sending emails from a server can be useful, either because applications require it, or to do *monitoring*, such as being notified of security updates available on your system or the proper execution of *cron* tasks.

However, configuring a mail server with `postfix`, `exim` or `sendmail` requires a large number of steps to be performed and a careful configuration, due to all the devices set up to fight spam{{% marginalia %}}including DKIM, SPF, DMARC, static IP and Reverse DNS, white list...{{% /marginalia %}}, which makes the task very tedious for a simple personal server.

Nevertheless, it is possible to simply use an existing email account, and send *via* SMTP emails as a traditional email client would. We will use *ssmtp* for this purpose.


### Installation of SSMTP

From a *Debian* based system, you can simply install the package with :

```
sudo apt-get install ssmtp
```

The configuration file is as follows:

```
sudo nano /etc/ssmtp/ssmtp.conf
```

Then replace the following settings with the IDs and settings of your email provider:

```bash
root=<your-email>
mailhub=<smtp-server>:<port>
hostname=<your-domain>
AuthUser=<username>
AuthPass=<password>
AuthMethod=LOGIN
UseTLS=Yes
UseSTARTTLS=Yes
```

For example, with Gmail:

```bash
root=<your-email>
mailhub=smtp.gmail.com:587
hostname=<your-domain>
AuthUser=<username>
AuthPass=<password>
AuthMethod=LOGIN
UseTLS=Yes
UseSTARTTLS=Yes
```

Or, with OVH :

```bash
root=<your-email>
mailhub=ssl0.ovh.net:587
hostname=<your-domain>
AuthUser=<username>
AuthPass=<password>
AuthMethod=LOGIN
UseTLS=Yes
UseSTARTTLS=Yes
```

Once the file has been saved, we can try to send our first emails.


### Test sending emails

It is possible to use `ssmtp` directly to send your email:

```bash
echo -e "Subject: Title\nMessage" | sudo ssmtp -vvv <email-adress>
```

If you receive a message like:

```
ssmtp: Cannot open <server>:<port>
```

This means that the connection settings to your email provider are incorrect, or that the port is not open.

The processes on your server will send emails directly using the `sendmail` command. To test it, you can use:

```bash
echo "Message" | sendmail -s "Title" <email-adress>
```

You can also test that the `mail` command works well:

```bash
echo "Message" | mail -s "Title" <email-adress>
```

If the two previous examples do not work, it may be necessary to create the link from `sendmail` to `ssmtp`:

```bash
sudo ln -s /usr/sbin/ssmtp /usr/sbin/sendmail
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