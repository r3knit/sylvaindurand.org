---
title: Emulate a browser <br/>with <em>Selenium</em>
categories: Python
---

Python is particularly effective when you want to automatically browse or perform actions on web pages. If it is enough to use libraries like [Beautiful Soup](https://en.wikipedia.org/wiki/Beautiful_Soup) when you just scrape web pages, it is sometimes necessary to perform actions on pages that require Javascript, or even to imitate human behavior. To do this, it is possible to fully emulate a web browser with Python.

## Installation

We will use *Selenium*, which is easily installed with :

```
pip install selenium
```

To be able to use it, it is necessary to install a rendering engine. We will use *Gecko*, the Firefox rendering engine.

To do this, go to [github.com/mozilla/geckodriver](https://github.com/mozilla/geckodriver/releases/) and download the latest version available for your platform. We decompress it, and place it in `/usr/local/bin`.

```
tar xvfz geckodriver-version-plateform.tar.gz
mv geckodriver /usr/local/bin
```


## Usage in Python

To use *Selenium*, simply import at the beginning of the file:

```python
from selenium import webdriver
```

### Simple usage

You can start *Selenium* with:

```python
driver = webdriver.Firefox(executable_path=r'/usr/local/bin/geckodriver')
```

Thus, to click on an element, we use for example:

```python
driver.find_element_by_class_name('my_class').click()
```

To free the memory, you can exit *Selenium* with :

```
driver.quit()
```


### Use without graphical interface

If you want to launch it without a graphical user interface, you can use:

```python
options = webdriver.FirefoxOptions()
options.add_argument('-headless')
driver = webdriver.Firefox(options=options, executable_path=r'/usr/local/bin/geckodriver')
```

### Using with a proxy or Tor

If you use a proxy, or *Tor* (which is used as a proxy, with local IP `127.0.0.0.1` and port `9050`), it is possible to connect to it with Selenium using the following options:

```python
profile = webdriver.FirefoxProfile()
profile.set_preference("network.proxy.type", 1)
profile.set_preference("network.proxy.socks", '127.0.0.1')
profile.set_preference("network.proxy.socks_port", 9050)
profile.set_preference("network.proxy.socks_remote_dns", False)
profile.update_preferences()
```

You can then use:

```python
driver = webdriver.Firefox(firefox_profile=profile, executable_path=r'/usr/local/bin/geckodriver')
```

### Cache and cookies

Other options are available, for example to disable the cache:

```python
profile.set_preference("browser.cache.disk.enable", False)
profile.set_preference("browser.cache.memory.enable", False)
profile.set_preference("browser.cache.offline.enable", False)
profile.set_preference("network.http.use-cache", False)
```

It is also possible to clear cookies with:

```python
driver.delete_all_cookies()
```
