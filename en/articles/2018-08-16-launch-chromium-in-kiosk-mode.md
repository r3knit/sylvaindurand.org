---
title: Launch Chromium in kiosk mode
alt: Launch *Chromium* <br> in kiosk mode
categories: Raspberry Pi
---

Chromium makes it easy to be started in "kiosk" mode, that is to say to launch in full screen, without any window border, toolbar or notification{{% marginalia %}}surprisingly, this feature is not offered by Mozilla Firefox{{% /marginalia %}}.

This makes it possible to display a presentation, or a page, on a standalone display or on a terminal... or, in my case, to [display a slideshow on my television](https://github.com/sylvaindurand/reddit-slideshow/) while the Raspberry Pi casts music.

Our goal will be to get:

* a *minimal* installation of the graphical server, without desktop environment or window manager;
* an automatic launch on startup, in full screen;
* no toolbar or notification.


## Display server

### Installation

To display the browser, we will have to install an [X server](https://en.wikipedia.org/wiki/X_Window_System). There is no need to install a desktop environment or window manager, unnecessarily large, since the browser will be launched directly in full screen.

```
sudo apt-get install xserver-xorg-video-all xserver-xorg-input-all xserver-xorg-core xinit x11-xserver-utils
```

### Launch at startup

To start the server automatically at startup, edit the `~/.bash_profile` file, which is executed when the user logs in, to put the following content{{% marginalia %}}the server starts with `startx`, but it is also necessary to check that a screen is available to avoid an error, for example, with SSH{{% /marginalia %}}:

```bash
if {{% marginalia %}} -z $DISPLAY {{% /marginalia %}} && {{% marginalia %}} $(tty) = /dev/tty1 {{% /marginalia %}}; then
    startx
fi
```

Your system must also be configured so that the user is automatically logged in at startup. How to proceed depends on your configuration {{% marginalia %}}see [this page](https://unix.stackexchange.com/questions/401759/automatically-login-on-debian-9-2-1-command-line) for Debian, [this one](https://askubuntu.com/questions/175248/how-to-autologin-without-entering-username-and-passwordin-text-mode) for Ubuntu or [that one](https://unix.stackexchange.com/questions/42359/how-can-i-autologin-to-desktop-with-systemd) for Arch{{% /marginalia %}}. With *Raspbian*, launch `raspi-config` utility, then choose `Boot Options` and `Console Autologin`.


## Chromium

### Installation

We will install, of course, Chromium, but also `unclutter`, which will allow us to hide the pointer of the mouse:

```
sudo apt-get install chromium-browser
sudo apt-get install unclutter
```

### Launch at startup

To start them automatically at startup, we create a file`~/.xinitrc`{{% marginalia %}}this file is executed when the X server starts{{% /marginalia %}} which contains the following commands (take care to choose your URL and indicate the screen resolution) :


```bash
#!/bin/sh
xset -dpms
xset s off
xset s noblank

unclutter &
chromium-browser /path/to/your/file.html --window-size=1920,1080 --start-fullscreen --kiosk --incognito --noerrdialogs --disable-translate --no-first-run --fast --fast-start --disable-infobars --disable-features=TranslateUI --disk-cache-dir=/dev/null  --password-store=basic
```

The `xset` commands are used to avoid the automatic standby of the system, which will otherwise interrupt the display after a specified time.

The option `--window-size=` is essential, otherwise Chromium will only be displayed on half of the screen, despite the other instructions, in the absence of a window manager.
