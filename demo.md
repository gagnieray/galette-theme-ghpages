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

## Horizontal rule

---

That's all of it.
