---
ref: home
title: Galette Theme
---

The GitHub Pages theme for [Galette](https://galette.eu) plugin websites. It
carries the same identity as galette.eu — PT Sans, the orange `#ffb619`, the
blue `#007baa` and the grey band behind the logo — and it states, on every page,
whether the plugin is maintained by the Galette team or by a community member.

See the [element reference](demo.html) for how content is rendered.

## Using it

Add this to the `_config.yml` on your plugin's `gh-pages` branch:

```yaml
remote_theme: galette/theme-ghpages
title: Galette Fullcard
description: Full member card as PDF
maintainer: core            # core | community
plugin:
  archive: galette-plugin-fullcard
  version: "2.2.1"
  min_galette: "1.3.0"
defaults:
  - scope: {path: ""}
    values: {layout: default, lang: en}
```

Then make sure GitHub Pages is set to build from the `gh-pages` branch, root
directory. Nothing else is needed: no workflow, and no `_layouts` of your own —
a local `_layouts/default.html` would shadow the theme's.

<div class="table-wrapper" markdown="1">

| Key | Required | What it does |
| --- | --- | --- |
| `title` | yes | Plugin name, shown as the page heading |
| `description` | yes | Tagline under the heading, and the meta description. A page may override it with its own `description` front matter — that is how a translated page gets a translated tagline. |
| `maintainer` | yes | `core` shows a “Maintained by the Galette team” pill outlined in Galette orange with an outlined star, `community` a “Community plugin” one outlined in grey with a group icon. Anything else is treated as `community`. |
| `tracker_url` | no | Bug tracker in the menu, defaults to the repository's GitHub issues |
| `default_lang` | no | Language the menu falls back to when a page has no translation, defaults to `en` |
| `author` | no | Name in the copyright line, defaults to Johan Cwiklinski |
| `galette_url` | no | Overrides the logo target |
| `galette_doc_url` | no | Overrides the “Galette documentation” menu entry |
| `galette_contact_url` | no | Overrides the “Get in touch” target |

</div>

The source repository link comes from `site.github.*`, so it needs no
configuration.

## The menu and the download cartouche

Two distinct things, and they are not interchangeable: the menu is the sidebar
list on the right, the cartouche is the orange box in the header.

The **menu** lists the site's own pages first, then the bug tracker and the
source repository, then a Galette block. On a small screen it folds into an
off-canvas panel driven by `:target`, so it works without JavaScript.

The **download cartouche** points at wherever the plugin's archives actually
live — releases on GitHub, nightly builds on galette.eu:

<div class="table-wrapper" markdown="1">

| Key | Required | What it does |
| --- | --- | --- |
| `plugin.archive` | no | Archive base name. With `plugin.version` it builds both URLs on galette.eu: `{archive}-{version}.tar.bz2` and `{archive}-dev.tar.bz2`. |
| `plugin.version` | no | Version shown on the stable button, `latest` when absent |
| `plugin.min_galette` | no | Minimum Galette version, shown beside the maintainer pill |
| `plugin.release_url` | no | Overrides the derived stable URL; falls back to the repository's `releases/latest` |
| `plugin.nightly_url` | no | Overrides the derived nightly URL |
| `plugin.name` | no | Label on both buttons, defaults to `title` |
| `galette_download_url` | no | Base the two URLs are built on, defaults to `https://galette.eu/download/plugins` |

</div>

The whole cartouche disappears when a site declares no download at all.

Galette plugin releases live on galette.eu rather than on GitHub — plugin-fullcard
has no GitHub release at all — so declaring `archive` and `version` is the normal
way, and the version then lives in exactly one place.

The version and the minimum Galette version belong here and nowhere else: a
number written into a page becomes a string a translator has to carry, in every
language, and has to be bumped in each of them at every release.


## Pages and languages

A page declares two things in its front matter, and nothing more:

```yaml
---
ref: doc          # stable across languages: home, doc, …
title: Documentation
---
```

The language comes from the file's directory, through path-scoped `defaults` —
never from the front matter. That matters: it lets Weblate write translated
Markdown files without ever having to produce a correct `lang` key.

```
index.md              -> /              lang: en   ref: home
documentation.md      -> /documentation  lang: en   ref: doc
fr/index.md           -> /fr/            lang: fr   ref: home
fr/documentation.md   -> /fr/documentation  lang: fr  ref: doc
```

Pages sharing a `ref` are offered to each other in the language picker — a
`<details>` disclosure showing the current language and folding the others away,
so it works with the keyboard and without JavaScript — and the menu keeps the
reader inside their language. Declare one `defaults` entry per language you
publish:

```yaml
defaults:
  - scope: {path: ""}
    values: {layout: default, lang: en}
  - scope: {path: "fr"}
    values: {lang: fr}
```

Do **not** add `permalink` to a translated page: the URL comes from the path, so
two languages can never collide on the same one.

## Adding a language to the interface

The theme knows the nineteen languages Galette translates into — the list shared
with the core, the manual and every plugin, on
[Weblate](https://hosted.weblate.org/projects/galette/).

Interface strings — the menu labels, the maintainer sentences, the cartouche, the
footer — live in `i18n/<lang>.yml` in the theme repository, which is what Weblate
translates; `bin/build-i18n` turns them into the `_includes` the theme actually
ships. Ten languages have their own strings today, the nine others render the
English ones while still declaring their own language and text direction.

Right-to-left languages get `dir="rtl"`, Galette's mirrored header photo, and a
layout built on logical properties.

Publishing a language on your own site is independent of that: add its
`defaults` entry and the pages, and the selector picks it up.

The eight strings the menu and the cartouche need are part of that set, so a new
language means one `when` branch in each of the two includes.

## Licence

The theme is GPL-3.0-or-later, like the galette.eu stylesheets it derives from.
Site contents are expected to be CC BY-SA 4.0, as stated in the footer.
