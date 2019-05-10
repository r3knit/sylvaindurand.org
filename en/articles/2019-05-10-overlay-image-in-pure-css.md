---
title: Overlay image in pure CSS
alt: Overlay image <br> in pure CSS
categories: CSS
---

There is no native solution within browsers to display, on click, a full screen image. Several *Javascript* libraries have come to fill this gap: [*Lightbox*](https://lokeshdhakar.com/projects/lightbox/){{% marginalia %}}which was a precursor, to the point of [giving its name to the concept](https://en.wikipedia.org/wiki/Lightbox_(JavaScript)){{% /marginalia %}}, [*Fancybox*](http://fancybox.net), [*PhotoSwipe*](https://photoswipe.com)...

These solutions are sometimes heavy, whereas if you are only looking for a simple behavior, a few lines of CSS are enough, and no Javascript code is required. Here is an example of what we will get at the end of this article (click on the image!):

{{% img src="/img/samples/egypt.jpg" alt="Nil" /%}}


This example, which contains only CSS and no Javascript code{{% marginalia %}}for only 400 bytes, compared to several tens or hundreds of Kb for the above-mentioned libraries{{% /marginalia %}} works on all recent browsers. Its simplicity should allow you to configure it as much as you want!

## Principle

### :target selector

Nous allons utiliser le sélecteur CSS `:target`, qui permet d'appliquer un style à un élément lorsque son identifiant (`id=""`) est celui de l'ancre (`#`) dans l'URL de la page. Pour prendre un exemple :

We will use the CSS selector `:target`, which allows to apply a style to an element when its identifier (`id=""`) is the one of the anchor (`#`) in the page URL. To take an example:


```html
<a href="#myid">Click to color in red!</a>
<div id="myid">Hello World!</div>

<style>
div:target {
  color: red;
}
</style>
```

Note that this selector actually allows you to add a lot of interactive elements to a page without Javascript, such as showing and hiding an element or a navigation bar.

### Compatibility

According to [developer.mozilla.org](https://developer.mozilla.org/en-US/docs/Web/CSS/:target) and [caniuse.com](https://caniuse.com/#feat=css-sel3), the `:target` selector is supported by all browsers released since 2008, and especially since Internet Explorer 9.


## HTML
For each image, the HTML markup will be as simple as possible:

```html
<!-- The link that, when clicked, will display the image in full screen -->
<a href="#img-id">
  <img src="image-thumbnail.png" alt="Thumbnail">
</a>

<!-- The full screen image, hidden by default  -->
<a href="#_" class="overlay" id="img-id">
  <img src="image-fullscreen.png" alt="Fullscreen">
</a>
```

The first line, customizable at will, will display the image in full screen thanks to the link to `#img-id`. The second, hidden by default, contains the corresponding identifier.

If several images are present on the page, it will obviously be necessary to give them a unique identifier for each one, and to match the link.

The link to `#_` has no particular meaning, and could have been replaced by `#anything`. It simply aims to cause `#img-id` to lose the status `:target` and therefore to close the full screen when clicking.


## CSS

### Display

The overlay is nothing more than a `div` with a fixed position, located at the top left and covering the entire page. We add a `flex` property to it to place the image in the center of it:

```css

.overlay {
  /* Display over the entire page */
  position: fixed;
  z-index: 99;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0,0,0,0.9);

  /* Horizontal and vertical centering of the image */
  display: flex;
  align-items: center;
  text-align: center;

  /* We hide all this by default */
  visibility: hidden;
}

.overlay img{
  /* Maximum image size */
  max-width: 90%;
  max-height: 90%;

  /* We keep the ratio of the image */
  width: auto;
  height: auto;
}
```

### Activation on click

As we have seen previously, we now have to add a `visibility: visible` with the selector `:target` to display all this when you click on the trigger link.


```css
.overlay:target {
  visibility: visible;
  outline: none;
  cursor: default;
}
```

We add an `outline: none' to avoid the border that is displayed, by default, on several browsers. In addition, `cursor: default`, optional, avoids having a "hand" in place of the cursor on the entire page.

### Animation

The *Javascript* libraries mentioned above allowed to have a nice fade effect when opening. Never mind: CSS allows almost any kind of madness in terms of animation.

We're going to stay sober here, with a slight bland effect, with a very small zoom effect on the image.

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

