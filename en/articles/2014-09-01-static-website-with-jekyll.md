---
title: Static website with Jekyll
alt: Static website <br> with *Jekyll*
categories: Jekyll
---

At the beginning of the Internet, there were *static sites*: each web page was written "by hand" using a text editor, and then put online. The disadvantages are many, especially the need to duplicate the same changes on some pages{{% marginalia %}}this is why it was sometimes really hard to maintain a website{{% /marginalia %}}, to know HTML and to have his computer available to edit pages. The advent of CSS, which allows to separate actual content from its presentation format and to share it between the pages has not changed this fact{{% marginalia %}}moreover, the inexistant interoperability between browsers and the poor support of CSS with Microsoft Internet Explorer, have delayed its use{{% /marginalia %}}.

It was then that appeared *dynamic sites*: the different programming languages, running on the server side, such as PHP, helped the rise of [CMS](https://en.wikipedia.org/wiki/Content_management_system), which made possible to create sites and change their content directly from a browser, thus allowing the emergence of sites, blogs, forums accessible to the greatest number. This is for example the case of [*Spip*](https://www.spip.net/), [*Dotclear*](https://dotclear.org/) or [*WordPress*](https://wordpress.com/). However, these systems are not without disadvantages:

- they are very sensitive to security gaps, which implies that you need to  monitor carefully updates and logs;
- they are server ressource intensive, and need specific hostings for large volume of visitors;
- they badly handle significant increases in workload, so they are very sensitive to DDoS attacks or huge influx of visitors{{% marginalia %}}it is not uncommon that a website become unavailable because of an important event or a link from a news website{{% /marginalia %}};
- they tend to be a labyrinthine system, largely overkill for their use and needing databases.

For a couple of years, *static websites* has come back into favor with the emergence of the *static website generators*. With simple text files, a program generates a website made entirely from static pages you just have to host. Security problems are thus virtually non-existent, it is possible to host your website on a very modest server or rather the opposite, to get excellent performances and handle huge increases in workload using a [CDN](https://en.wikipedia.org/wiki/Content_delivery_network) like [*Cloudflare*](https://www.cloudflare.com/) or [*Cloudfront*](https://aws.amazon.com/cloudfront/){{% marginalia %}}ways to host a static website on [*Amazon S3*](https://aws.amazon.com/s3/) and [*Cloudfront*](https://aws.amazon.com/cloudfront/) are explained in "[Static website with *Cloudfront*](/static-website-with-cloudfront/)"{{% /marginalia %}}.

In addition, it is possible to follow the changes and to work collaboratively thanks to `git`, to write the articles online and to generate the website on the fly with services like [*GitHub*](https://pages.github.com/) and [*Prose*](https://prose.io), or to have a commenting system with [*Disqus*](https://disqus.com/).

This article will show how to install (I) and use (II) the [*Jekyll*](https://jekyllrb.com/) website static generator to create and modify a simple website.

## First website with *Jekyll*

### Installation of *Jekyll*

Directly install the last stable version of *Ruby*, or [RVM](https://rvm.io/), then simply launch:

```
gem install jekyll
```

*On Windows*, it is less easy to install *Jekyll*; however, [Julian Thilo](https://jekyll-windows.juthilo.com) wrote a [very detailed guide](https://jekyll-windows.juthilo.com) about how to do it.

### Creation of a new website

The command `jekyll new mysite` will create the source code of a working website in the `mysite` folder. In this folder, you can generate the website with `jekyll build`. The output can then be seen in the `_site/` folder.

With a single command, it is possible to generate the website and create a local host in order to watch the produced website: use `jekyll serve` in order to get your website available on `http://localhost:4000`. You can also automatically regenerate the website each time you change something in the source code{{% marginalia %}}however, this option doesn't detect the changes provided in `_config.yml`{{% /marginalia %}} with `jekyll serve -w`, which will be by far the most useful command when you'll start to play with *Jekyll*.

### Tree structure

*Jekyll* uses several folders:

- *`_posts/`* in which the articles are stored{{% marginalia %}}you can freely organize your files in `_post`{{% /marginalia %}}, with names such as `yyyy-mm-dd-post-name.md`;
- *`_layouts/`* which contains the layout of the website, that is to say everythin that will surround the articles;
-  *`_includes/`*, which contains small page fragments that you wish to include in multiple places on your site{{% marginalia %}}when you put a file in `_includes`, it is possible to include it anywhere with the tag `{{ include filename }}`. Il is also possible to provide it [parameters](https://jekyllrb.com/docs/templates/#includes){{% /marginalia %}}.

The tree structure may then looks like:

```
mysite/
    _includes/
    _layouts/
        default.html
        page.html
        post.html
    _posts/
        2014-08-24-static-website-with-jekyll.md
    assets/
        style.sass
        script.js
        favicon.ico
    index.html
    rss.xml
    _config.yml
```

You can add any folder or file in your website folder. If they don't start with an underscore, *Jekyll* will generate them on the same location.

## Using *Jekyll*

Now that the first website is created, we will see how to make it evolve, how to write articles and use metadata.

In order to create an article, juste create in `_posts` folder a file with a name with the following format: `yyyy-mm-dd-post-name.md`{{% marginalia %}}it is also possible to create articles in the folder `_drafts`, without any date in the file name: thus, it will create drafts invisible in the posts list but available with their URL{{% /marginalia %}}. This file is divided into two sections: the *frontmatter* where the metadata are stored, and the *content* of the article.

### Frontmatter and metadata

The frontmatter allows us to declare metadata, which will be called or tested in the website. It is set into the top of the file, under the following format:

```
---
layout: default
title: My title
---
```

Only the `layout` variable is required: it defines which file in `_layouts/` *Jekyll* should use to build the page. It is also usual to define a `title` variable in order to provide a title to our article{{% marginalia %}}some variables are reserved by *Jekyll* with a particuliar behaviour: `permalink` for exemple specifies the final URL of a file{{% /marginalia %}}.

It is also possible to define default variables, declared once for all or parts of the articles. For instance, instead of declare `layout: default` in each article stored in `_posts/`, you may declare it once in `_config.yml`:

```
defaults:
  -
    scope:
      path: "_posts"
    values:
      layout: "default"
```

### Writing articles with *Markdown*

By default, *Jekyll* use [*Markdown*](https://daringfireball.net/projects/markdown/basics). The purpose of this language is to provide a very simple syntax to replace the most commons HTML tags. For example, `*italic*` gives "*italic*" and `**bold**` gives "**bold**". It is however still quite possible to use HTML in posts.

From its second version, *Jekyll* uses *Kramdown* which add many features like the possibility of giving CSS classes to elements, footnotes, definition lists, tables...

### Using metadata

Any metadata "`variable`", declared in the frontmatter or as a default, can be called anywhere in the website, with the tag `{{ page.variable }}`, which returns its value.

It is also possible to do some tests:

```
{% if page.variable == 'value' %}
    banana
{% else %}
    coconut
{% endif %}
```

We can also, for example, make loops on each article satisfying some conditions:

```
{% assign posts=site.posts | where: "variable", "value" %}
{% for post in posts %}
    {{ post.lang }}
{% endfor %}
```

Although the [syntax](https://github.com/Shopify/liquid/wiki/Liquid-for-Designers) may not be very elegant to use, the large number of [available variables](https://jekyllrb.com/docs/variables/), plus the custom metadata, combined with the [filters et commands](https://github.com/Shopify/liquid/wiki/Liquid-for-Designers), may become highly effective.

---

### And much more...

This article doesn't claim to be more than a very short introduction to *Jekyll*. To go further, read the [*Jekyll* excellent documentation](https://jekyllrb.com/docs/home/) first, regularly updated, and the numerous references you will find on the web.

You may also read three other articles written on this website about *Jekyll*:

- [creating a multilingual website](/making-jekyll-multilingual/) as it has been done here;
- [website delivery with *CloudFront*](/website-delivery-with-cloudfront/) in order to get excellent performances and handle huge increases in workload;
- [hosting *Jekyll* on GitHub](/using-github-to-serve-jekyll/) in order to edit your website online and generate it on the fly.

Lastly, browsing [website using *Jekyll* source codes](https://github.com/jekyll/jekyll/wiki/Sites){{% marginalia %}}feel free to browse the [source code of this website](https://github.com/sylvaindurand/sylvaindurand.org) to find out how it works{{% /marginalia %}}, in order to find inspiration, can only be a good idea.
