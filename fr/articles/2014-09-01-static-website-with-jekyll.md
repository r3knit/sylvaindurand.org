---
title: Site statique avec Jekyll
alt: Site statique <br> avec *Jekyll*
aliases: site-statique-avec-jekyll
categories: Jekyll
---

Au commencement d'Internet étaient les *sites statiques* : chaque page web était rédigée "à la main" à l'aide d'un éditeur de texte, puis mise en ligne. Les inconvénients étaient nombreux, en particulier la nécessité de dupliquer une même modification sur chaque page concernée{{% marginalia %}}il en découlait une  réelle difficulté à faire évoluer un site sur le long terme{{% /marginalia %}}, de connaître le HTML pour le rédacteur et d'avoir son ordinateur à disposition pour éditer les pages. L'apparition du CSS, permettant de séparer la forme du fond et de mutualiser celle-ci pour l'ensemble d'un même site n'a guère changé cet état de fait{{% marginalia %}}de surcroît, la très faible interopérabilité entre navigateurs et le très mauvais support de CSS par Microsoft Internet Explorer, alors très dominant, ont fortement retardé son utilisation{{% /marginalia %}}.

C'est alors que sont apparus les *sites dynamiques* : les langages de programmation exécutés côté serveur, tels que PHP, ont permis de voir apparaître les [CMS](https://fr.wikipedia.org/wiki/Syst%C3%A8me_de_gestion_de_contenu), qui rendaient possible la création de sites et la modification de leur contenu directement depuis un navigateur, permettant ainsi l'émergence de sites, blogs, forums accessibles au plus grand nombre. C'est par exemple le cas de [*Spip*](https://www.spip.net/), [*Dotclear*](https://fr.dotclear.org/) ou [*WordPress*](https://fr.wordpress.com/). Pourtant, ces systèmes ne sont pas dénués d'inconvénients :

- ils sont très sensibles aux failles de sécurité, ce qui implique de surveiller attentivement les mises à jour et les logs ;
- ils sont consommateurs de ressources serveur, nécessitant des hébergements spécifiques pour les gros volumes de visiteurs ;
- ils supportent mal les montées en charge, et sont ainsi très sensibles aux attaques DDoS ou aux affluences de visiteurs{{% marginalia %}}il n'est pas rare qu'un site devienne indisponible, par exemple lors d'un événement important ou en raison d'un lien publié sur un site d'actualités{{% /marginalia %}} ;
- ils constituent souvent de vraies usines à gaz, surdimensionnées vis-à-vis des besoins et nécessitant des bases de données.

Depuis quelques années, les *sites statiques* font leur retour en grâce avec l'apparition des *générateurs de sites statiques*. Sur la base de simples fichiers textes, un programme génère un site composé uniquement de pages statiques qu'il suffit ensuite d'héberger. Ainsi, les problèmes de sécurité sont presque inexistants, il est possible de s'héberger sur un serveur très modeste ou au contraire d'obtenir d'excellentes performances et de supporter de très fortes montées en charge en utilisant un [CDN](https://fr.wikipedia.org/wiki/Content_delivery_network) comme [*Cloudflare*](https://www.cloudflare.com/) ou [*Cloudfront*](https://aws.amazon.com/fr/cloudfront/){{% marginalia %}}les façons d'héberger un site statique sur [*Amazon S3*](https://aws.amazon.com/fr/s3/) et [*Cloudfront*](https://aws.amazon.com/fr/cloudfront/) sont détaillés dans "[Site statique avec *Cloudfront*](/site-statique-avec-cloudfront/)"{{% /marginalia %}}.

Il est de plus possible de suivre toutes les modifications et de travailler collaborativement grâce à `git`, de rédiger ses articles en ligne et de générer son site à la volée à l'aide de services comme [*GitHub*](https://pages.github.com/) et [*Prose*](https://prose.io), ou d'avoir un système de commentaires avec [*Disqus*](https://disqus.com/).

Dans cet article, nous allons voir comment installer (I) et utiliser (II) le générateur de site statiques [*Jekyll*](https://jekyllrb.com/) pour créer et modifier un site simple.

## Premier site avec *Jekyll*

### Installation de *Jekyll*

Pour installer Jekyll, après avoir installé *Ruby*, par exemple avec [RVM](https://rvm.io/), lancez simplement la commande suivante :

```
gem install jekyll
```

*Sur Windows*, l'installation est moins aisée ; [Julian Thilo](https://jekyll-windows.juthilo.com/) a écrit un [guide très détaillé](https://jekyll-windows.juthilo.com/) sur les façons d'y installer *Jekyll*.

### Création d'un nouveau site

La commande `jekyll new monsite` permet d'obtenir le code source d'un site fonctionnel dans le dossier `monsite`. Dans ce dossier, vous pouvez le générer avec `jekyll build`. Le site est alors créé dans `_site/`.

En une seule commande, il est possible de générer le site et de créer un serveur local pour visualiser le site produit : utilisez `jekyll serve` pour pouvoir l'observer à l'adresse `http://localhost:4000`. Il est également possible de regénérer le site à chaque modification du code source{{% marginalia %}}cette option ne prend cependant pas pas en compte les modifications de `_config.yml`{{% /marginalia %}} avec `jekyll serve -w`, qui sera de loin la commande la plus utile lorsque vous utiliserez régulièrement *Jekyll*.

### Arborescence

Le code source d'un site *Jekyll* s'organise selon plusieurs dossiers :

- *`_posts/`* dans lequel seront placés{{% marginalia %}}l'arborence interne du dossier `_post` est laissée entièrement libre{{% /marginalia %}} tous les articles de votre site, au format `aaaa-mm-jj-nom-du-post.md` ;
- *`_layouts/`* qui va contenir la maquette du site, c'est-à-dire tout ce qui entourera nos articles ;
-  *`_includes/`*, qui contiendra des codes que vous pouvez inclure{{% marginalia %}}en plaçant un fichier dans `_includes`, il vous sera possible de l'importer n'importe où avec `{{ include nom-du-fichier }}` ; il est même possible de lui passer des [paramètres](https://jekyllrb.com/docs/templates/#includes){{% /marginalia %}} dans différentes pages si vous en avez besoin régulièrement.

Le répertoire de votre site pourra alors ressembler à :

```
monsite/
    _includes/
    _layouts/
        default.html
        page.html
        post.html
    _posts/
        2014-08-24-site-statique-avec-jekyll.md
    assets/
        style.sass
        script.js
        favicon.ico
    index.html
    rss.xml
    _config.yml
```

Vous pouvez ajouter n'importe quel autre dossier, ou fichier, dans le répertoire de votre site. Tant qu'ils ne commencent pas par un tiret bas, ceux-ci seront directement générés au même emplacement par *Jekyll*.

## Utilisation de *Jekyll*

Maintenant ce premier site créé, nous allons voir comment le faire évoluer en rédigeant des articles et en utilisant leurs métadonnées.

Pour créer un article, il suffit de créer dans le dossier `_posts` un fichier dont le nom est au format `aaaa-mm-jj-nom-du-post.md`{{% marginalia %}}il est également possible de créer des articles dans le dossier `_drafts` sans date dans le nom de fichier : cela permet de créer des brouillons d'article, qui n'apparaîtront pas dans la liste des articles disponibles mais restent accessibles depuis leur adresse directe{{% /marginalia %}}. Ce fichier se compose de deux parties : l'*en-tête* où sont situées les métadonnées de l'article, et le *contenu* de l'article à proprement parler.

### Déclarer les métadonnées des fichiers

L'en-tête permet de déclarer les métadonnées de l'article ; celles-ci pourront par la suite être appelées ou testées dans le reste du site. Elle est présente en début de fichier, sous la forme suivante :

```
---
layout: default
title: Mon titre
---
```

Seule la variable `layout` est obligatoire : elle définit le fichier que *Jekyll* doit utiliser dans le dossier `_layouts/` pour construire la page autour de l'article. Il est également usuel de définir `title` qui permet de définir le titre de l'article{{% marginalia %}}il existe malgré tout des variables spécifique à *Jekyll* dont le rôle est particulier : `permalink` permet par exemple d'indiquer l'adresse à laquelle l'article sera accessible{{% /marginalia %}}.

Il est également possible de définir des variables "par défaut" qui concerneront tous les articles d'un dossier. Par exemple, pour ne pas avoir à indiquer `layout: default` au sein de tous les articles placés dans `_posts/`, il est possible de la définir par défaut dans `_config.yml` :

```
defaults:
  -
    scope:
      path: "_posts"
    values:
      layout: "default"
```

### Écriture des articles avec *Markdown*

Par défaut, les articles s'écrivent en [*Markdown*](https://daringfireball.net/projects/markdown/basics). L'objectif de ce langage est de proposer une syntaxe très simple permettant de rédiger les articles en évitant les balises HTML les plus courantes. Ainsi, "*italique*" s'obtient avec `*italique*`, et "**gras**" avec `**gras**`. Il reste cependant toujours possible d'utiliser HTML au sein des articles.

Depuis sa deuxième version, *Jekyll* utilise *Kramdown* qui ajoute de nombreuses fonctionnalités telles que la possibilité d'attribuer des classes aux éléments, les notes de bas de page, les listes de définition, les tableaux...

### Utilisation des métadonnées

Toute métadonnée "`variable`" déclarée dans l'entête peut être appelée, dans n'importe quel fichier, à l'aide d'une balise `{{ page.variable }}` qui retournera alors sa valeur.

Il est également possible d'effectuer des tests :

```
{% if page.variable == 'value' %}
    banane
{% else %}
    noix de coco
{% endif %}
```

Nous pouvons aussi, par exemple, effectuer des boucles sur l'ensemble des articles répondant à certaines conditions :

```
{% assign posts=site.posts | where: "variable", "value" %}
{% for post in posts %}
    {{ post.lang }}
{% endfor %}
```

Bien que la [syntaxe](https://github.com/Shopify/liquid/wiki/Liquid-for-Designers) ne soit pas toujours très élégante à utiliser, le grande nombre de [variables disponibles](https://jekyllrb.com/docs/variables/), auxquelles s'ajoutent les métadonnées personnalisées que vous créerez ainsi que les nombreux [filtres et commandes](https://github.com/Shopify/liquid/wiki/Liquid-for-Designers), peuvent être extrêmement efficaces.

---

### Et bien plus...

Cet article n'a pas prétention à constituer davantage qu'une très brève introduction à *Jekyll*. Pour personnaliser votre site davantage, lisez en priorité l'[excellente documentation de *Jekyll*](https://jekyllrb.com/docs/home/), bien tenue à jour, ainsi que les nombreuses références que vous trouverez sur Internet.

Vous pouvez également consulter sur ce site trois autres articles à propos de *Jekyll* :

- [créer un site multilingue avec *Jekyll*](/rendre-jekyll-multilingue/) comme cela a été réalisé ici ;
- [servir son site à l'aide de *CloudFront*](/servir-son-site-avec-cloudfront/) pour obtenir des performances maximales en termes de disponibilité et de vitesse ;
- [héberger _Jekyll_ sur GitHub](/utiliser-github-pour-servir-jekyll/) pour pouvoir suivre et modifier votre site en ligne, en le générant à la volée.

Enfin, parcourir les [codes sources de sites utilisant *Jekyll*](https://github.com/jekyll/jekyll/wiki/Sites){{% marginalia %}}vous êtes notamment libres de consulter le [code source du présent site](https://github.com/sylvaindurand/sylvaindurand.org) pour voir comment celui-ci est conçu{{% /marginalia %}}, pour vous inspirer, ne peut être qu'une excellente idée.
