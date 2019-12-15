---
title: Statistiques de mails Outook avec Python
alt: Statistiques de mails <br>Outlook avec *Python*
categories: Python
---

Le logiciel de messagerie Microsoft Outlook utilise un format de données propriétaires, le « PST »{{% marginalia %}}*Personal Storage Table*{{% /marginalia %}}, pour stocker les emails, des rendez-vous ou des tâches. Il a fallu attendre 2010 pour que les spécifications de ce format soient rendues publiques, ce qui explique probablement que peu d'outils soient disponibles pour l'ouvrir et en exploiter les données.

Sous Python, il existe cependant la librairie [*libpff*](https://github.com/libyal/libpff) qui permet d'en exporter la plupart des métadonnées. Les lignes suivantes montrent comment l'utiliser pour faire un graphique des emails reçus.

## Installation de *libpff*

Comme toutes les librairies Python, *libpff* peut être installé avec `pip` :

```
pip install libpff-python
```

Cependant, à l'heure de la rédaction de ces lignes, la version proposée par *pip*{{% marginalia %}}la version 20161119{{% /marginalia %}} ne permet pas de récupérer l'heure des messages. Des correctifs ont été apportés dans une version plus récente{{% marginalia %}}la version 20190725{{% /marginalia %}}, accessible avec :

```
pip install libpff-python-ratom
```


## Récupération des mails

### Ouverture du fichier

On commence par charger la librairie :
```python
import pypff
```

Puis on ouvre notre fichier : l'ouverture peut néanmoins être assez longue selon la taille de votre archive.
```python
pst = pypff.file()
pst.open("mails.pst")
```

### Extraction des métadonnées

Il est possible de naviguer dans la structure en utilisant les fonctions offertes par la librairie, depuis la racine :

```python
root = pst.get_root_folder()
```

Pour extraire les données, on peut alors utiliser une fonction récursive :

```python
def parse_folder(base):
    messages = []
    for folder in base.sub_folders:
        if folder.number_of_sub_folders:
            messages += parse_folder(folder)
        print(folder.name)
        for message in folder.sub_messages:
            messages.append({
                "subject": message.subject,
                "sender": message.sender_name,
                "datetime": message.client_submit_time
            })
    return messages

messages = parse_folder(root)
```

Cette fonction peut être assez lente selon le nombre de messages (de l'ordre de 300 messages sont traités par seconde sur mon ordinateur).

Une fois cela fait, peut alors importer ce fichier dans *Pandas* :

```python
import pandas as pd
df = pd.DataFrame(messages)
```

### Gestion de l'heure

Les heures extraites du fichier sont stockées au format UTC, ce qui signifie que vous allez devoir retraiter le bon fuseau horaire. On commence par déclarer le fuseau horaire UTC, avant de le convertir au fuseau horaire souhaité :

```python
df['datetime'] = df['datetime'].dt.tz_localize(tz='UTC')
df['datetime'] = df['datetime'].dt.tz_convert(tz='Europe/Paris')
```

### Exemple de graphique

Nous allons tracer un nuage de points affichant l'heure d'arrivée des emails selon la date. Pour cela, on crée deux colonnes avec les coordonnées des points à tracer :

```python
df['hour'] = df['datetime'].dt.hour + df['datetime'].dt.minute / 60
df['date'] = df['datetime'].dt.year + df['datetime'].dt.dayofyear / 365
```

On trace alors :

```python
import matplotlib.pyplot as plt
import seaborn as sns

plt.clf()
ax = sns.scatterplot(x="date", y="hour", s=3, alpha=.3, linewidth=0, marker=".", data=df)
ax.set(xlim=(2014.5,2020), ylim=(7,25))
ax.invert_yaxis()
sns.despine()
ax.get_figure().savefig("plot.png", dpi=400)
```

Cela nous donne :

![Nuage de points](/img/stats/mails.png)
