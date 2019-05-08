---
title: Utiliser Tor sous Python
alt: Utiliser *Tor* <br> sous *Python*
categories: Python
---

Cette page vous montrera comment utiliser *Tor* pour accéder, en tout anonymat, à des données à l'aide d'un script *Python*. Cela peut être particulièrement utile si vous réalisez un *scrapper* sans se faire bannir par le serveur concerné.



## Installation de *Tor*

L'installation de *Tor* dépend de votre système, et est détaillée sur le [site officiel](https://www.torproject.org). Sur un système *Debian* ou *Raspbian* qui en est son dérivé, on utilise alors :

```
sudo apt-get install tor
```

Pour lancer *Tor*, il suffit de lancer :

```
sudo service tor start
```

Pour vérifier qu'il fonctionne, il suffit de lancer la commande suivante depuis un terminal :

```
curl --socks5 localhost:9050 --socks5-hostname localhost:9050 -s https://check.torproject.org/ | cat | grep -m 1 Congratulations | xargs
```

Il faut que cette commande affiche :

```
Congratulations. This browser is configured to use Tor.
```


## Utilisation sous *Python*

### Utilisation avec *requests*
Pour demander une page, utilisons la librarie `requests`. Si vous ne l'avez pas, il suffit de l'installer :

```
pip install requests
pip install requests[socks]
pip install requests[security]
```

S'il y a une erreur pour l'installation de `cryptography` la commande précédente, installez les paquets suivants :

```
sudo apt-get install build-essential libssl-dev libffi-dev python-dev
```


On utilise alors, dans Python :

```python
import requests
```

Vous pouvez vérifier votre adresse IP sans *Tor* avec la commande :

```python
requests.get('https://ident.me').text
```

Pour utiliser *Tor*, on lui dit d'utiliser un proxy :

```python
proxies = {
    'http': 'socks5://127.0.0.1:9050',
    'https': 'socks5://127.0.0.1:9050'
}

requests.get(url, proxies=proxies).text
```

Ainsi, vous devriez avoir une nouvelle adresse IP avec :

```python
requests.get('https://ident.me', proxies=proxies).text
```

### Obtention d'une nouvelle identité

Si vous avez besoin d'une nouvelle identité, et de changer d'adresse IP, il faut installer *stem* :

```
pip install stem
```

Il faut aussi configurer le controlleur de *Tor* pour pouvoir demander le renouvellement :

```
sudo nano /etc/tor/torrc
```

On utilise les paramètres :

```
ControlPort 9051
CookieAuthentication 1
```

Puis on redémarre *Tor* pour prendre en compte ces modifications :

```
sudo service tor restart
```

Depuis *Python*, il suffit alors d'utiliser la commande suivante :

```python
from stem import Signal
from stem.control import Controller

with Controller.from_port(port = 9051) as c:
    c.authenticate()
    c.signal(Signal.NEWNYM)
```

Pour le vérifier, on regarde si l'on obtient une nouvelle IP avec :

```python
requests.get('https://api.ipify.org', proxies=proxies).text
```

### Renforcer l'anonymat en changeant l'*User-Agent*

Si l'anonymat est requis, il peut être utile de changer l'*[user-agent](https://fr.wikipedia.org/wiki/User_agent)*, qui trahit notre identité auprès du serveur. On installe pour cela `fake_useragent` :

```
pip install fake_useragent
```

On peut alors utiliser, dans Python :

```python
from fake_useragent import UserAgent
headers = { 'User-Agent': UserAgent().random }
requests.get(url, proxies=proxies, headers=headers).text
```

### Automatisation avec *Cron*

Si votre script Python est amené à être utilisé régulièrement à l'aide d'une tâche [Cron](https://fr.wikipedia.org/wiki/Cron), il peut être utile de rajouter un délai aléatoire pour éviter que l'heure d'accès ne soit trop régulière :

```python
import random, time
wait = random.uniform(0, 2*60*60)
time.sleep(wait)
```

