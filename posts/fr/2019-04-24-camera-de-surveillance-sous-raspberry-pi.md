---
title: Caméra de surveillance <br/> sous Raspberry Pi
categories: Raspberry Pi
---

Si les caméras de surveillance connectées peuvent être bien utiles, elles sont également susceptibles de présenter de graves failles de sécurité. Elles sont ainsi à l'origine de l'[une des plus importantes attaques DDOS](http://www.lefigaro.fr/secteur/high-tech/2016/10/24/32001-20161024ARTFIG00180-comment-des-cameras-de-surveillance-ont-reussi-a-paralyser-une-partie-du-web.php), et certaines sont parfois même [visibles publiquement](http://www.insecam.org/). De fait, il est bien difficile d'identifier l'origine des caméras ; aucune garantie ne peut être apportée sur le niveau de sécurité ni la pérénité des serveurs du fabricant.

Pourtant, il est très simple – et très économique – de parvenir à réaliser soi-même un simple système de surveillance à l'aide d'un Raspberry Pi. La fondation Raspberry Pi propose une petite [caméra](https://www.raspberrypi.org/products/camera-module-v2/), de qualité très correcte. De nombreuses autres marques existent, et l'article suivant fonctionnera également avec une simple webcam USB.

![Raspberry Pi camera]({{ site.data.translations[page.lang].base }}/assets/raspberry/raspberrycamera.jpg)


## Activation de la caméra

Commencez par raccorder la caméra au Raspberry Pi en utilisant la [bonne interface](https://projects.raspberrypi.org/en/projects/getting-started-with-picamera/4). Appliquez-vous lors de ce branchement : j'ai mis longtemps à essayer de faire fonctionner cette caméra avant de réaliser que la nappe n'était pas insérée jusqu'au fond du connecteur.

Il est ensuite nécessaire d'activer la caméra avec :

```
sudo raspi-confi
```

Sélectionnez `Interfacing Options` » `P1 Camera` » `Enable`, avant de redémarrer le Raspberry Pi.

La caméra utilise le pilote `v4l2`, intégré par défaut dans le Kernel fourni avec Raspbian. Pour tester si la caméra fonctionne bien, vous pouvez prendre une photo avec :

```
raspistill -v -o test.jpg
```

On peut également enregistrer une vidéo (ici pendant cinq secondes) :

```
raspivid -o test.h264 -t 5000
```

Si votre caméra est à l'envers pour des raisons de branchement, il est possible de la retourner avec :

```
v4l2-ctl --set-ctrl vertical_flip=1
v4l2-ctl --set-ctrl horizontal_flip=1
```

## Premier essai : MotionEye

J'ai dans un premier temps essayé d'utiliser l'excellent logiciel [MotionEye](https://github.com/ccrisan/motioneye), qui offre une interface web permettant de voir la webcam, mais aussi de faire des enregistrements et de détecter des mouvements.

La procédure d'installation est très bien détaillée sur le [site officiel](https://github.com/ccrisan/motioneye/wiki/Install-On-Raspbian).

Je ne suis cependant pas parvenu à obtenir un *framerate* convenable : quel que soit les réglages utilisés (y compris avec une résolution et une qualité minimales), je n'ai jamais réussi à dépasser 1.0 fps, ce qui rend le résultat assez décevant...



## Deuxième essai : v4l2rtspserver

Le bien nommé RTSP, pour *Real Time Streaming Protocol*, permet de mettre en place un streaming particulièrement fluide.

### Installation

Le logiciel [v4l2rtspserver](https://github.com/mpromonet/v4l2rtspserver) permet d'utiliser simplement ce protocole avec notre caméra. Vous pouvez l'installer avec :

```
sudo apt-get install cmake liblog4cpp5-dev libv4l-dev
git clone https://github.com/mpromonet/v4l2rtspserver.git
cd v4l2rtspserver/
cmake .
make
sudo make install
```

### Configuration

On peut alors lancer un flux vidéo avec :

```
v4l2rtspserver -H 972 -W 1296 -F 15 -P 8555 /dev/video0
```

La commande précédente permet de délibrer une image en 1296x972 (voir les différents [modes conseillés](https://picamera.readthedocs.io/en/release-1.12/fov.html)), avec 15 fps (qui permet une luminosité convenable dans une pièce sombre).

Le flux vidéo devient alors disponible depuis l'adresse suivante depuis tout logiciel pouvant le lire, comme par exemple VLC (sur votre ordinateur comme sur téléphone) :

```
rtsp://<raspberry-pi-ip>:8555/unicast
```

L'image devient parfaitement fluide et, dès lors que la connexion est suffisante, la qualité est très convenable. Il subsiste en revanche un temps de latence d'environ une seconde.

Il est à noter que [v4l2rtspserver](https://github.com/mpromonet/v4l2rtspserver) permet également de spécifier un nom d'utilisateur et un mot de passe, pour plus de sécurité.


### Lancement au démarrage

Pour lancer v4l2rtspserver avec le démarrage, commençons par inscrire la commande avec les paramètres ci-dessus dans la section `ExecStart` du fichier suivant :

```
sudo nano /lib/systemd/system/v4l2rtspserver.service
```

On active alors v4l2rtspserver avec le démarrage :

```
sudo systemctl enable v4l2rtspserver
```
