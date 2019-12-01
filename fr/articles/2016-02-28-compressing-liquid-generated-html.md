---
title: Compresser le code généré par Liquid
alt: Compresser le code <br> généré par *Liquid*
aliases: compresser-le-code-genere-par-liquid
categories: Jekyll
---

La syntaxe *Liquid*, notamment utilisée par *Jekyll*, a le désagréable défaut de générer des quantités importantes d'espaces et de sauts de ligne inutiles dans le code source de notre page.

C'est particulièrement vrai lorsque l'on souhaite indenter ses codes *Liquid*, ou que l'on utilise des boucles.

Par exemple, le code suivant{{% marginalia %}}qui affiche les multiplications qui ont pour résultat "12"{{% /marginalia %}} va générer près de 500 lignes de code, presque toutes vides:

```liquid
<ul>
{% for i in (1..12) %}
  {% for j in (1..12) %}
  {% assign result = i | times: j %}
  {% if result == 12 %}
    <li> {{ i }} ⨉ {{ j }} = 12 </li>
  {% endif %}
  {% endfor %}
{% endfor %}
</ul>
```

Que se passe-t-il ? *Liquid* lit l'ensemble des caractères présents dans la boucle, y compris les espaces et les sauts de ligne, et les restitue sans se poser de questions. Cette double boucle génère donc des centaines de sauts de lignes et d'espaces inutiles.

Cependant, pour avoir un code HTML plus lisible, nous préférerions comme résultat :

```html
<ul>
    <li> 1 ⨉ 12 = 12 </li>
    <li> 2 ⨉ 6 = 12 </li>
    <li> 3 ⨉ 4 = 12 </li>
    <li> 4 ⨉ 3 = 12 </li>
    <li> 6 ⨉ 2 = 12 </li>
    <li> 12 ⨉ 1 = 12 </li>
</ul>
```

## Première piste : désindenter son code

La première solution, qui est aussi la moins satisfaisante, est de supprimer les espaces et sauts de ligne que l'on ne souhaite pas voir affichés. Dans l'exemple précédent, nous obtiendrions alors :

```liquid
<ul>{% for i in (1..12) %}{% for j in (1..12) %}{% assign result = i | times: j %}{% if result == 12 %}
    <li> {{ i }} ⨉ {{ j }} = 12 </li>{% endif %}{% endfor %}{% endfor %}
</ul>
```

Le code source généré est impeccable, mais notre code *Liquid* est pratiquement illisible...

## Deuxième piste : utiliser `capture`

La balise `capture` permet de stocker, sous forme de variable, ce qui est généré en son sein.

En utilisant un `capture` autour d'un code *Liquid*, on masque tout ce qu'il produira, notamment les espaces et sauts de ligne indésirés. Il suffit alors d'un deuxième `capture` autour de la partie intéressante, que l'on affiche ensuite !

Le précédent code source deviendrait ainsi :

```liquid
<ul>{% capture hide %}
{% for i in (1..12) %}
  {% for j in (1..12) %}
  {% assign result = i | times: j %}
  {% if result == 12 %}
{% capture show %}{{ show }}
    <li> {{ i }} ⨉ {{ j }} = 12 </li>{% endcapture %}
  {% endif %}
  {% endfor %}
{% endfor %}
{% endcapture %}{{ show }}
</ul>
```

Le code reste cependant un peu verbeux, et n'est pas très élégant.

## Utiliser  `%}{%` , la solution ?

Depuis *Jekyll* 3.0.0, il est possible d'insérer des sauts de lignes et des espaces *dans* les balises *Liquid*, qui n'auront aucun effet sur le code source généré. Il est donc possible d'identer son code ainsi :

```liquid
<ul>{%
for i in (1..12) %}{%
  for j in (1..12) %}{%
    assign result = i | times: j %}{%
    if result == 12 %}
    <li> {{ i }} ⨉ {{ j }} = 12 </li>{%
    endif %}{%
  endfor %}{%
endfor %}
</ul>
```

Cette façon d'écrire, bien qu'inhabituelle, reste très lisible et permet d'atteindre le résultat voulu.

Cependant, je n'ai pas réussi à trouver si ce comportement est une fonctionnalité souhaitée des développeurs de *Liquid*, et donc s'il sera maintenu dans le temps.

## Compresser l'ensemble du code HTML généré

Jetez un coup d'œil à la méthode développée par [penibelst](https://jch.penibelst.de), très astucieuse et qui comporte de nombreuses options.
