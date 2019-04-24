---
title: Surveillance camera <br/> with Raspberry Pi
categories: Raspberry Pi
---

While connected surveillance cameras can be useful, they are also likely to have serious security breaches. They are thus at the origin of [one of the most important DDOS attacks](https://www.forbes.com/sites/thomasbrewster/2016/09/25/brian-krebs-overwatch-ovh-smashed-by-largest-ddos-attacks-ever/), or are sometimes [publicly visible](http://www.insecam.org/). In fact, it is very difficult to identify the origin of the cameras; no guarantee can be given on the security level or the durability of the manufacturer's servers.

However, it is very simple – and very economical – to be able to make a simple monitoring system yourself using a Raspberry Pi. The Raspberry Pi Foundation offers a small [camera module]((https://www.raspberrypi.org/products/camera-module-v2/)). Many other brands exist, and the following article will also work with a simple USB webcam.

![Raspberry Pi camera]({{ site.data.translations[page.lang].base }}/assets/raspberry/raspberrycamera.jpg)


## Camera activation

First connect the camera to the Raspberry Pi using the [correct interface](https://projects.raspberrypi.org/en/projects/getting-started-with-picamera/4). Apply yourself to this connection: it took me a long time to try to operate this camera before I realized that the web was not inserted to the bottom of the connector.

It is then necessary to activate the camera with:

```
sudo raspi-confi
```

Select `Interfacing Options` » `P1 Camera` » `Enable`, before restarting the Raspberry Pi.

The camera uses the `v4l2` driver, which is integrated by default in the kernel provided with Raspbian. To test if the camera is working properly, you can take a picture with :

```
raspistill -v -o test.jpg
```

You can also record a video (here for five seconds):

```
raspivid -o test.h264 -t 5000
```

If your camera is upside down for connection reasons, you can return it with :

```
v4l2-ctl --set-ctrl vertical_flip=1
v4l2-ctl --set-ctrl horizontal_flip=1
```

## First try: MotionEye

I first tried to use the excellent [MotionEye](https://github.com/ccrisan/motioneye) software, which offers a web interface to view the webcam, but also to make recordings and detect movements.

The installation procedure is very well detailed on the [official website](https://github.com/ccrisan/motioneye/wiki/Install-On-Raspbian).


However, I couldn't get a suitable framerate: whatever the settings used (including with minimal resolution and quality), I never managed to exceed 1.0 fps, which makes the result quite disappointing...


## Second try: v4l2rtspserver

The well named RTSP, for *Real Time Streaming Protocol*, allows to set up a particularly fluid streaming.

### Installation

The [v4l2rtspserver](https://github.com/mpromonet/v4l2rtspserver) software allows you to easily use this protocol with our camera. You can install it with:

```
sudo apt-get install cmake liblog4cpp5-dev libv4l-dev
git clone https://github.com/mpromonet/v4l2rtspserver.git
cd v4l2rtspserver/
cmake .
make
sudo make install
```

### Configuration

You can then start a video stream with:

```
v4l2rtspserver -H 972 -W 1296 -F 15 -P 8555 /dev/video0
```

The previous command allows you to delimit an image in 1296x972 (see the different [recommended modes](https://picamera.readthedocs.io/en/release-1.12/fov.html)), with 15 fps (which allows a suitable brightness in a dark room).

The video stream then becomes available from the following address from any software that can play it, such as VLC (on your computer or phone):

```
rtsp://<raspberry-pi-ip>:8555/unicast
```

The image becomes perfectly smooth and, as soon as the connection is sufficient, the quality is very suitable. However, there is still a latency time of about one second.

It should be noted that v4l2rtspserver also allows you to specify a user name and password, for greater security.


### Launch on startup

To launch v4l2rtspserver at boot, let's start by entering the command with the above parameters in the `ExecStart` section of the following file:

```
sudo nano /lib/systemd/system/v4l2rtspserver.service
```

We then activate v4l2rtspserver at boot:

```
sudo systemctl enable v4l2rtspserver
```
