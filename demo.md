---
ref: doc
title: Element reference
---

Every element the theme styles, on one page, so a change can be eyeballed at a
glance. The French version of this page exercises the language selector.

## Headings

### Third level

#### Fourth level

## Text

A paragraph with **bold**, *italic*, `inline code`, a [link to galette.eu](https://galette.eu)
and a footnote-free sentence long enough to wrap on a narrow screen so the
justified alignment can be judged honestly.

> A blockquote, for the notes and warnings a plugin documentation always ends up
> needing.

## Lists

* first item
* second item, with a nested list:
  * nested one
  * nested two
* third item

1. numbered
2. numbered again

## Download badges

<div class="download-badge" markdown="1">
[![Get the latest release](https://img.shields.io/badge/2.2.1-Fullcard-ffb619.svg?logo=php&logoColor=white&style=for-the-badge)](https://galette.eu/download/plugins/)
[![Get the nightly build](https://img.shields.io/badge/nightly-Fullcard-ffb619.svg?logo=php&logoColor=white&style=for-the-badge)](https://galette.eu/download/plugins/)
</div>

## Code

```bash
$ cd /var/www/html/galette/plugins
$ wget https://galette.eu/download/plugins/galette-plugin-fullcard-2.2.1.tar.bz2
$ tar xjvf galette-plugin-fullcard-2.2.1.tar.bz2
```

## Table

Wide tables scroll inside their own container, so the page itself never scrolls
sideways — wrap them in `<div class="table-wrapper" markdown="1">`.

<div class="table-wrapper" markdown="1">

| Setting | Default | Description |
| --- | --- | --- |
| `maintainer` | `community` | Which pill the header shows |
| `plugin.archive` | — | Base name of the release archive |
| `plugin.min_galette` | — | Minimum supported Galette version |

</div>

## Admonitions

Write one as a blockquote whose first word is bold. Nothing else is needed, and
that is the point: a page Weblate rewrites can hold no Liquid tag, since a tag a
translation breaks fails the whole build.

```markdown
> **Warning** — check the usage policy of the provider you choose.
```

> **Warning** — Check the usage policy of the provider you choose. Most of them
> are run by associations or by volunteers, and they set conditions on the
> traffic they accept.

> **Note** — The provider setting appeared in version 2.3.0.

> **Todo** — Document the second map layer.

The word carries the colour, and it is matched in every language the theme
knows, because it is part of the translated paragraph:

> **Avertissement** — Vérifiez la politique d'utilisation du fournisseur que
> vous choisissez.

A word no catalogue lists still gets a box, only a neutral one, so a translation
nobody anticipated degrades instead of losing its styling:

> **Attention** — a word the catalogues do not carry, so this keeps the neutral
> box instead of losing its styling.

A quotation that merely contains bold text stays a quotation — which is why the
test lives in the script and not in a CSS `:has()`, where `:first-child` would
match this one too:

> Ceci est une citation ordinaire, **avec du gras** au milieu.

The `alert.html` include is still there for a page no translator touches:

{% include alert.html type="todo" content="This is a todo" %}

{% include alert.html type="note" content="This is a note" %}

{% include alert.html type="warning" content="This is a warning" %}

## Images

An image the content column has to shrink becomes clickable, and opens at its
full size. Nothing is written in the Markdown to ask for it: the script compares
the picture's natural width to the width it is given, which is the automatic
equivalent of the `:scale:` the Sphinx manual relies on.

The same file is used twice here on purpose, so the viewer has two images to move
between — arrows, a counter, and the arrow keys.

![A screenshot wider than the column that holds it](assets/images/og_image.jpg)

![The very same file, so the viewer has somewhere to go](assets/images/og_image.jpg)

The caption in the viewer is the alt text, which Weblate already translates. An
image small enough to be shown whole gains nothing from a viewer, so it is left
alone — this one is 208px wide and stays an ordinary image:

![The Galette logo, too small to be worth opening](assets/images/galette.png)

## Horizontal rule

---

That's all of it.
