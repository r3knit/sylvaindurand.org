---
title: Émuler un navigateur <br/>avec <em>Selenium</em>
categories: Python
---

Python est particulièrement efficace lorsque l'on souhaite parcourir ou effectuer des actions de façon automatisée sur des pages web. S'il suffit d'utiliser des librairies comme [Beautiful Soup](https://fr.wikipedia.org/wiki/Beautiful_Soup) lorsque l'on se contente de [scraper](https://fr.wikipedia.org/wiki/Web_scraping) des pages web, il est parfois nécessaire d'effectuer des actions sur des pages qui requierent Javascript, voire d'imiter un comportement humain. Pour cela, il est possible d'émuler intégralement un navigateur web avec Python.

## Installation

Nous allons ici utiliser *Selenium*, qui s'installe facilement avec :

```
pip install selenium
```

Pour pouvoir l'utiliser, il est nécessaire d'installer un moteur de rendu. Nous allons ici utiliser *Gecko*, le moteur de rendu de Firefox.

Pour cela, allez sur [github.com/mozilla/geckodriver](https://github.com/mozilla/geckodriver/releases/) et téléchargez-y la dernière version disponible pour votre plateforme. On le décompresse, et on le place dans ` /usr/local/bin`.

```
tar xvfz geckodriver-version-plateform.tar.gz
mv geckodriver /usr/local/bin
```


## Utilisation dans Python

Pour utiliser *Selenium*, il suffit d'importer en début de fichier :

```python
from selenium import webdriver
```

### Utilisation simple

On peut lancer *Selenium* avec :

```python
driver = webdriver.Firefox(executable_path=r'/usr/local/bin/geckodriver')
```

Ainsi, pour cliquer sur un élément, on utilise par exemple :

```python
driver.find_element_by_class_name('my_class').click()
```

Pour libérer la mémoire, on peut quitter *Selenium* avec :

```
driver.quit()
```


### Utilisation sans interface graphique

Si l'on veut le lancer sans interface graphique, on utilise :

```python
options = webdriver.FirefoxOptions()
options.add_argument('-headless')
driver = webdriver.Firefox(options=options, executable_path=r'/usr/local/bin/geckodriver')
```

### Utilisation avec un proxy ou avec Tor

Si vous utilisez un proxy, ou *Tor* (qui s'utilise comme un proxy, avec l'IP locale `127.0.0.1` et le port `9050`), il est possible de s'y connecter avec Selenium en utilisant les options suivantes :

```python
profile = webdriver.FirefoxProfile()
profile.set_preference("network.proxy.type", 1)
profile.set_preference("network.proxy.socks", '127.0.0.1')
profile.set_preference("network.proxy.socks_port", 9050)
profile.set_preference("network.proxy.socks_remote_dns", False)
profile.update_preferences()
```

Vous pouvez alors utiliser :

```python
driver = webdriver.Firefox(firefox_profile=profile, executable_path=r'/usr/local/bin/geckodriver')
```

### Cache et cookies

D'autres options sont disponibles, par exemple pour désactiver le cache :

```python
profile.set_preference("browser.cache.disk.enable", False)
profile.set_preference("browser.cache.memory.enable", False)
profile.set_preference("browser.cache.offline.enable", False)
profile.set_preference("network.http.use-cache", False)
```

Il est par ailleurs possible de vider les cookies avec :

```python
driver.delete_all_cookies()
```
