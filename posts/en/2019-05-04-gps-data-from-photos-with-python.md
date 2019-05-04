---
title: GPS data from <br/> photos with <em>Python</em>
categories: Python
---


Most recent cameras and mobile phones record photographs with geographical data (longitude, latitude, but also altitude). If this data is readable with most photo and file explorer software, it is also possible to access it with Python.

Here we will see how to access all the [EXIF](https://en.wikipedia.org/wiki/Exif) metadata of an image, then decode the GPS data. For the example, we will use the following photograph, taken in [Þórsmörk](https://en.wikipedia.org/wiki/Thórsmörk) in Iceland:

{:.large}
[![Þórsmörk]({{ site.data.translations[page.lang].base }}/assets/gps/thorsmork_small.jpg)]({{ site.data.translations[page.lang].base }}/assets/gps/thorsmork.jpg)


## Reading metadata

The *Python Imaging Library* (PIL) provides easy access to data[EXIF] (https://en.wikipedia.org/wiki/Exif) with the function `_getexif()`.

It is easily installed, under Python3, with the *fork*[Pillow] (https://pillow.readthedocs.io). We use the following command[[depending on your configuration, `pip3` can be directly accessible with `pip`]]:

```
pip3 install pillow
```

However, this results in an indexed dictionary with numerical identifiers. To get the corresponding names, we use `ExifTags` and rename the keys of the dictionary:

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

The `exif` variable then contains all the metadata of the image: objective, aperture, speed, author... Our GPS data is stored in `exif['GPSInfo']`, in the following form:

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

## Getting GPS data

### Sexagesimal form

Geographic coordinates are usually expressed in the sexagesimal system, or DMS for degrees (°), minutes (′) and seconds (″). The unit is the degree of angle (1 turn = 360°), then the minute of angle (1° = 60′), then the second of angle (1° = 3 600″).

Compared to the equatorial plane, the latitude is completed by a letter `N` (hemisphere) or `S` depending on whether one is located in the northern or southern hemisphere. Compared to the Greenwich meridian, the longitude is completed by a letter `W` or `E` depending on whether you are located in the west or east.

The data is directly accessible in `exif['GPSInfo']` to be able to output this format:

```python
def get_coordinates(info):
    for key in ['Latitude', 'Longitude']:
        if 'GPS'+key in info and 'GPS'+key+'Ref' in info:
            e = info['GPS'+key]
            ref = info['GPS'+key+'Ref']
            info[key] = ( str(e[0][0]/e[0][1]) + '°' +
                                 str(e[1][0]/e[1][1]) + '′' +
                                 str(e[2][0]/e[2][1]) + '″ ' +
                                 ref)

    if 'Latitude' in info and 'Longitude' in info:
        return [info['Latitude'], info['Longitude']]

get_coordinates(exif['GPSInfo'])
```

We get:

```js
['63.0°40.0′59.08″ N', '19.0°31.0′51.39″ W']
```


### Decimal form

To obtain automated geographic data processing, a decimal format is often more convenient: minutes are divided by 60 and seconds by 3600 and added together. Latitude is negative in the Southern Hemisphere (S), and west of the Greenwich Meridian (W). The calculation is then very simple:

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

We get:

```js
[63.683077777777775, -19.530941666666667]
```

## Avec GPSPhoto

Since the end of 2018, the Python library `GPSPhoto` allows to obtain directly the geographical coordinates of a photo.

It is installed with:

```
pip3 install GPSPhoto
```

We can now use:

```python
from GPSPhoto import gpsphoto
gpsphoto.getGPSData('Þórsmörk.jpg')
```

We get:

```js
{'Latitude': 63.683077777777775,
 'Longitude': -19.530941666666667,
 'Altitude': 485.3874345549738,
 'UTC-Time': '13:18:42',
 'Date': '09/03/2018'}
```

Nevertheless, according to the file, GPSPhoto tends to easily return errors that, in my case, make it impossible to automate a large number of photos:

```python
ValueError: malformed node or string: <_ast.BinOp object at 0x10ea16320>
```

## Automation

If you want to recover metadata from a set of photos, you can iterate to a folder (and its subfolders).

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

You can then export `points` to display it on a map, for example with[Leaflet](https://leafletjs.com).
