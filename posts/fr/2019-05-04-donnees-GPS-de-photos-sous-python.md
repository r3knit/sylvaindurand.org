---
title: Données GPS de <br/> photos avec <em>Python</em>
categories: Python
---


La plupart des appareils photos récents et téléphones portables enregistrent les photographies avec des données géographiques (longitude, latitude, mais aussi altitude). Si ces données sont lisibles avec la majorité des logiciels photos et d'explorateurs de fichiers, il est également possible d'y accéder avec Python.

Nous allons ici voir comment accéder à l'ensemble des métadonnées [EXIF](https://fr.wikipedia.org/wiki/Exchangeable_image_file_format) d'une image, puis y décoder les données GPS. Pour l'exemple, nous allons utiliser la photographie suivante, prise à [Þórsmörk](https://fr.wikipedia.org/wiki/Þórsmörk) en Islande :

{:.large}
[![Þórsmörk]({{ site.data.translations[page.lang].base }}/assets/gps/thorsmork_small.jpg)]({{ site.data.translations[page.lang].base }}/assets/gps/thorsmork.jpg)


## Lecture des métadonnées


La bibliothèque *Python Imaging Library* (PIL) permet facilement d'accéder aux données [EXIF](https://fr.wikipedia.org/wiki/Exchangeable_image_file_format) avec la fonction `_getexif()`.

On l'installe facilement, sous Python3, avec le *fork* [Pillow](https://pillow.readthedocs.io). On utilise la commande suivante[[selon votre configuration, `pip3` peut être directement accessible avec `pip`]] :

```
pip3 install pillow
```

Cependant, on obtient par ce biais un dictionnaire indexé avec des identifiants numériques. Pour avoir les noms correspondants, on utilise `ExifTags` et on renomme les clefs du dictionnaire :

```python
from PIL import Image
from PIL.ExifTags import TAGS, GPSTAGS

def get_exif(filename):
    exif = Image.open(filename)._getexif()

    if exif is not None:
        for key, value in exif.items():
            name = TAGS.get(key, key)
            exif[name] = exif.pop(key)

        if 'GPSInfo' in exif:
            for key in exif['GPSInfo'].keys():
                name = GPSTAGS.get(key,key)
                exif['GPSInfo'][name] = exif['GPSInfo'].pop(key)

    return exif

exif = get_exif('Þórsmörk.jpg')
```

La variable `exif` contient alors l'ensemble des métadonnées de l'image : objectif, ouverture, vitesse, auteur... Nos données GPS sont stockées dans `exif['GPSInfo']`, sous la forme suivante :

```js
{'GPSLongitude': ((19, 1), (31, 1), (5139, 100)),
 'GPSAltitudeRef': b'\x00',
 'GPSAltitude': (92709, 191),
 'GPSTimeStamp': ((13, 1), (18, 1), (42, 1)),
 'GPSHPositioningError': (10, 1),
 'GPSLatitudeRef': 'N',
 'GPSLatitude': ((63, 1), (40, 1), (5908, 100)),
 'GPSLongitudeRef': 'W'}
 ```

## Récupération des données GPS

### Sous forme sexagésimale

Les coordonnées géographiques sont habituellement exprimées dans le système sexagésimal, ou DMS pour degrés (°), minutes (′) et secondes (″). L'unité  est le degré d'angle (1 tour = 360°), puis la minute d'angle (1° = 60′), puis la seconde d'angle (1° = 3 600″).

Par rapport au plan équatorial, la latitude est complétée d'une lettre `N` (hémisphère) ou `S` selon qu'on se situe dans l'hémisphère Nord ou  Sud. Par rapport au méridien de Greenwich, la longitude est complétée d'une lettre `W` ou `E` selon qu'on se situe à l'Ouest ou à l'Est.

Les données sont directement accessibles dans `exif['GPSInfo']` pour pouvoir sortir ce format :

```python
def get_coordinates(info):
    for key in ['Latitude', 'Longitude']:
        if 'GPS'+key in info and 'GPS'+key+'Ref' in info:
            e = info['GPS'+key]
            ref = info['GPS'+key+'Ref']
            info[key] = ( e[0][0]/e[0][1] +
                          e[1][0]/e[1][1] / 60 +
                          e[2][0]/e[2][1] / 3600
                        ) * (-1 if ref in ['S','W'] else 1)

    if 'Latitude' in info and 'Longitude' in info:
        return [info['Latitude'], info['Longitude']]

get_coordinates(exif['GPSInfo'])
```

On obtient :
```js
['63.0°40.0′59.08″ N', '19.0°31.0′51.39″ W']
```


### Sous forme décimale

Pour obtenir un traitement automatisé des données géographiques, un format décimal est souvent plus pratique : on divise les minutes par 60 et les secondes par 3600 et on additionne le tout. La latitude est négative dans l'hémisphère Sud (S), et à l'Ouest du méridien de Greenwich (W). Le calcul est alors tout simple :

```python
def get_decimal_coordinates(info):
    for key in ['Latitude', 'Longitude']:
        if 'GPS'+key in info and 'GPS'+key+'Ref' in info:
            e = info['GPS'+key]
            ref = info['GPS'+key+'Ref']
            info[key] = ( e[0][0]/e[0][1] +
                                 e[1][0]/e[1][1] / 60 +
                                 e[2][0]/e[2][1] / 3600
                               ) * (-1 if ref in ['S','W'] else 1)

    if 'Latitude' in info and 'Longitude' in info:
        return [info['Latitude'], info['Longitude']]

get_decimal_coordinates(exif['GPSInfo'])
```

On obtient :

```js
[63.683077777777775, -19.530941666666667]
```

## Avec GPSPhoto

Depuis la fin 2018, la bibliothèque Python `GPSPhoto` permet d'obtenir directement les coordonnées géographiques d'une photo. 

On l'installe avec :

```
pip3 install GPSPhoto
```

Il suffit alors d'utiliser :

```python
from GPSPhoto import gpsphoto
gpsphoto.getGPSData('Þórsmörk.jpg')
```

On obtient :

```js
{'Latitude': 63.683077777777775,
 'Longitude': -19.530941666666667,
 'Altitude': 485.3874345549738,
 'UTC-Time': '13:18:42',
 'Date': '09/03/2018'}
```

Néanmoins, selon le fichier, GPSPhoto a tendance à facilement renvoyer des erreurs qui, dans mon cas, rendent impossible toute automatisation sur un grand nombre de photos :

```python
ValueError: malformed node or string: <_ast.BinOp object at 0x10ea16320>
```

## Automatisation

Si l'on veut récupérer les métadonnées sur un ensemble de photos, il est possible d'itérer sur un dossier (et ses sous-dossiers).

```python
import os
points = []
for r, d, f in os.walk(path):
    for file in f:
        if file.lower().endswith(('.png', '.jpg', '.jpeg')):
            filepath = os.path.join(r, file)
            exif = get_exif(filepath)
            if exif is not None and 'GPSInfo' in exif:
                latlong = get_decimal_coordinates(exif['GPSInfo'])
                if latlong is not None:
                    points.append(latlong)
```

On peut alors exporter `points` pour l'afficher sur une carte, par exemple avec [Leaflet](https://leafletjs.com).
