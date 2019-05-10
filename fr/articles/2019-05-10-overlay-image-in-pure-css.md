---
title: Overlay d'une image en pur CSS seulement
alt: <em>Overlay</em> d'une image <br> en pur CSS
categories: CSS
---

Il n'existe pas de solution native au sein des navigateurs pour afficher, au clic, une image en plein écran. Plusieurs bibliothèques *Javascript* sont venues palier ce manque : [*Lightbox*](https://lokeshdhakar.com/projects/lightbox/){{% marginalia %}}qui fut précurseur, au point de [donner son nom au concept](https://en.wikipedia.org/wiki/Lightbox_(JavaScript)){{% /marginalia %}}, [*Fancybox*](http://fancybox.net), [*PhotoSwipe*](https://photoswipe.com)...

Ces solutions sont parfois lourdes, alors que si l'on ne recherche qu'un comportement simple, quelques lignes de CSS suffisent, et aucun code Javascript n'est nécessaire. Voici un exemple de ce que nous allons obtenir à la fin de cet article (cliquez sur l'image !) :

{{% img src="/img/samples/egypt.jpg" alt="Nil" /%}}


Cet exemple, qui ne contient que du CSS et aucun code Javascript{{% marginalia %}}pour seulement 400 octets, contre plusieurs dizaines ou centaines de Ko pour les librairies précitées{{% /marginalia %}}, fonctionne sur l'ensemble des navigateurs récents. Sa simplicité devrait vous permettre de le configurer autant que souhaité !

## Le principe

### Le sélecteur :target

Nous allons utiliser le sélecteur CSS `:target`, qui permet d'appliquer un style à un élément lorsque son identifiant (`id=""`) est celui de l'ancre (`#`) dans l'URL de la page. Pour prendre un exemple :

```html
<a href="#myid">Cliquez pour colorer en rouge !</a>
<div id="myid">Bonjour !</div>

<style>
div:target {
  color: red;
}
</style>
```

Notons que ce sélecteur permet en réalité d'ajouter énormément d'éléments interactifs sur une page sans Javascript, comme afficher et masquer un élément ou une barre de navigation.

### Compatibilité

D'après [developer.mozilla.org](https://developer.mozilla.org/en-US/docs/Web/CSS/:target) et [caniuse.com](https://caniuse.com/#feat=css-sel3), le sélecteur `:target` est supporté par tous les navigateurs sortis depuis 2008, et notamment depuis Internet Explorer 9.


## HTML
Pour chaque image, le code HTML sera on ne peut plus simple :

```html
<!-- Le lien qui, au clic, affichera l'image en plein écran -->
<a href="#img-id">
  <img src="image-thumbnail.png" alt="Vignette">
</a>

<!-- L'image en plein écran, cachée par défaut -->
<a href="#_" class="overlay" id="img-id">
  <img src="image-fullscreen.png" alt="Plein écran">
</a>
```

La première ligne, personnalisable à l'envie, affichera l'image en plein écran grâce au lien vers `#img-id`. La seconde, cachée par défaut, comporte l'identifiant correspondant.

Si plusieurs images sont présentes sur le page, il conviendra évidemment de leur donner un identifiant unique à chacune, et à mettre le lien en correspondance.

Le lien vers `#_` n'a pas de signification particulière, et aurait pu être remplacé par `#nimportequoi`. Il vise simplement à faire perdre le statut `:target` à `#img-id` et donc à fermer le plein écran au clic.


## CSS

### Affichage

L'overlay n'est rien d'autre qu'une `div` au positionnement fixé, située en haut à gauche et qui couvre l'ensemble de la page. Nous lui ajoutons une propriété `flex` pour placer l'image au centre de celle-ci :

```css

.overlay {
  /* Affichage par dessus l'ensemble de la page */
  position: fixed;
  z-index: 99;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0,0,0,0.9);

  /* Centrage horizontal et vertical de l'image */
  display: flex;
  align-items: center;
  text-align: center;

  /* On masque tout cela par défaut */
  visibility: hidden;
}

.overlay img{
  /* Taille maximale de l'image */
  max-width: 90%;
  max-height: 90%;

  /* On conserve le ratio de l'image */
  width: auto;
  height: auto;
}
```

### Affichage au clic

Comme nous l'avons vu précédemment, il suffit désormais d'ajouter un `visibility: visible` avec le sélecteur `:target` pour afficher tout cela lorsque l'on cliquera sur le lien déclencheur.

```css
.overlay:target {
  visibility: visible;
  outline: none;
  cursor: default;
}
```

Nous rajoutons un `outline: none` pour éviter la bordure qui s'affiche, par défaut, sur plusieurs navigateurs. Par ailleurs, `cursor: default`, optionnel, permet d'éviter d'avoir une "main" à la place du curseur sur l'ensemble de la page.

### Ajout d'une animation

Les bibliothèques *Javascript* citées précédemment permettaient d'avoir un joli effet de fondu à l'ouverture. Qu'à cela ne tienne : CSS permet à peu près toutes les folies en termes d'animation.

Nous allons rester sobre ici, avec un léger effet de fade, avec un tout petit effet de zoom sur l'image.

```css
.overlay {
  opacity: 0;
  transition: opacity .3s;
}

.overlay img {
  transform: scale(0.95);
  transition: transform .3s;
}

.overlay:target img {
    transform: scale(1);
}
```

### Code final

Notre code CSS est donc devenu :


### Full code:

Finally, our CSS code became:


```css

.overlay {
  position: fixed;
  z-index: 99;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0,0,0,0.9);
  display: flex;
  align-items: center;
  text-align: center;
  visibility: hidden;
  opacity: 0;
  transition: opacity .3s;
}

.overlay img{
  max-width: 90%;
  max-height: 90%;
  width: auto;
  height: auto;
  transform: scale(0.95);
  transition: transform .3s;
}

.overlay:target {
  visibility: visible;
  outline: none;
  cursor: default;
}

.overlay:target img {
    transform: scale(1);
}
```


## Bonus : shortcode pour *Hugo*

À l'aide du générateur de site statique *Hugo*, il est facile d'automatiser l'insertion à l'aide des [shortcodes](https://gohugo.io/content-management/shortcodes/). On créé pour cela un fichier `img.html` dans `layouts/shortcodes/` :

```html
<a href="#{{ anchorize (.Get "src") }}">
  <img src="{{.Get "src" }}" alt="{{.Get "alt" }}" />
</a>
<a href="#_" class="overlay" id="{{ anchorize (.Get "src") }}">
  <img src="{{.Get "src" }}" alt="{{.Get "alt" }}">
</a>
{{ .Inner }}
```

Il suffit alors, dans vos articles, d'inclure le shortcode :

```go
{{</* img src="image.jpg" alt="Description" /*/>}}
```

