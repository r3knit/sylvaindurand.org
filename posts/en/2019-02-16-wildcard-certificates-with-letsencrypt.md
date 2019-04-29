---
title: <em>Wildcard</em> certificates<br/> with Let’s Encrypt
original: 2018-03-17
---

Launched in late 2015, *[Let’s Encrypt](https://letsencrypt.org)* is a public benefit organization which democratized the use of HTTPS by providing free SSL certificates with an automated validation system.


After several months of testing, Let's Encrypt has launched the second version of its client ([ACME v2](https://community.letsencrypt.org/t/acme-v2-production-environment-wildcards/55578)) with a highly anticipated feature : you can obtain wildcard certificates, valid for all subdomains of one domain.

This possibility is particularly interesting when using many subdomains: so far, it was necessary to issue a new certificate to add a new subdomain, or to delete an old one. Here, a simple `*.domain.tld` is enough!


### Installation

To get our certificates, we will use the `certbot` client. The site offers several [installation methods](https://certbot.eff.org) depending on your platform. On Debian, you can simply use:


```
sudo apt-get install letsencrypt
```

### Issuing a certificate

Be careful, if you want a certificate for both the domain root (`domain.tld`) and its subdomains (`*.domain.tld`), both must be specified. With the `-d` parameter, it is possible to list the desired domains and subdomains:

```
sudo letsencrypt certonly --manual --preferred-challenges dns --register -d domain.tld -d *.domain.tld
```



### Validation

*Let's encrypt* will now have to ask us to prove that we have control over the domain names requested. It will request the creation of a specific TXT record in the DNS zone of the domain name, which can be done from your registar:

```
-------------------------------------------------------------------------------
Please deploy a DNS TXT record under the name
_acme-challenge.domain.tld with the following value:

c81US66r6JVk1LwyFHbzINQvIU_m5gJWXgcUm8Qj2

Before continuing, verify the record is deployed.
-------------------------------------------------------------------------------
Press Enter to Continue
```

Two TXT records will be requested for the same domain, it is completely normal (for both `domain.tld` and `*.domain.tld`).

Once created, the certificate is located in `/etc/letsencrypt/live/domain.tld`.




### Renewing certificates

In order to renew certificates periodicaly, we create a cron task:

```sudo crontab -e```

For instance, if you want to renew the certificates every Monday night, then restart nginx, we put:

```
30 4 * * 1 letsencrypt renew
40 4 * * 1 /etc/init.d/nginx reload
```
