---
title: Use <em>Tor</em><br/> with <em>Python</em>
categories: Python
---

This page will show you how to use *Tor* to anonymously access data with a *Python* script. This can be particularly useful if you want to create a *scrapper* without being banned by the server concerned.


## *Tor* installation

The installation of *Tor* depends on your system, and is detailed on the [official website](https://www.torproject.org). On a *Debian* or *Raspbian*, we use:

```
sudo apt-get install tor
```

To launch *Tor*, just run:

```
sudo tor start
```

To check if it works, simply run the following command from a terminal:

```
curl --socks5 localhost:9050 --socks5-hostname localhost:9050 -s https://check.torproject.org/ | cat | grep -m 1 Congratulations | xargs
```

This command will display:

```
Congratulations. This browser is configured to use Tor.
```


## Usage with *Python*

### With *requests* library
To request a page, use the `requests` library. If you do not have it, just install it:


```
pip install requests
pip install requests[socks]
pip install requests[security]
```

We then use:

```python
import requests
```

You can check your IP address without *Tor* with the command:

```python
requests.get('https://api.ipify.org').text
```

To use *Tor*, we tell it to use a proxy:

```python
proxies = {
    'http': 'socks5://127.0.0.1:9050',
    'https': 'socks5://127.0.0.1:9050'
}

requests.get(url, proxies=proxies).text
```

So, you should have a new IP address with:

```python
requests.get('https://api.ipify.org', proxies=proxies).text
```

### Obtaining a new identity

If you need a new identity, and change your IP address, you need to install *stem*:

```
pip install stem
```

The *Tor* controller must also be configured to request identity renewal:

```
sudo nano /etc/tor/torrc
```

We use the parameters:

```
ControlPort 9051
CookieAuthentication 1
```

Then we restart *Tor* to take into account these modifications:

```
sudo service tor restart
```

With *Python*, we now use the following command:

```python
from stem import Signal
from stem.control import Controller

with Controller.from_port(port = 9051) as c:
    c.authenticate()
    c.signal(Signal.NEWNYM)
```

To check it, we look if we get a new IP with:

```python
requests.get('https://api.ipify.org', proxies=proxies).text
```

### Strengthen anonymity by changing the *User-Agent*

If anonymity is required, it may be useful to change the *[user-agent](https://en.wikipedia.org/wiki/User_agent)* , which betrays our identity to the server. To do this, install `fake_useragent`:

```python
pip install fake_useragent
```

We can then use:

```python
from fake_useragent import UserAgent
headers = { 'User-Agent': UserAgent().random }
requests.get(url, proxies=proxies, headers=headers).text
```

### Automation with *Cron*

If your Python script is to be used regularly using a [Cron](https://en.wikipedia.org/wiki/Cron) job, it may be useful to add a random delay to prevent the access time from being too regular:


```python
import random, time
wait = random.uniform(0, 2*60*60)
time.sleep(wait)
```

