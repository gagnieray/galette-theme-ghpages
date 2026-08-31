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
  version: "2.2.1"
  release_url: https://github.com/galette-plugins/plugin-fullcard/releases/tag/2.2.1
  nightly_url: https://galette.eu/download/plugins/galette-plugin-fullcard-dev.tar.bz2
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
| `maintainer` | yes | `core` shows the orange “Maintained by the Galette team” pill, `community` the grey one. Anything else is treated as `community`. |
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
| `plugin.version` | no | Version shown on the stable button, `latest` when absent |
| `plugin.release_url` | no | Stable download, defaults to the repository's releases page |
| `plugin.nightly_url` | no | Nightly download; omit the button by leaving both this and `plugin.archive` unset |
| `plugin.archive` | no | Archive base name, a shortcut that builds `https://galette.eu/download/plugins/{archive}-dev.tar.bz2` |
| `plugin.name` | no | Label on both buttons, defaults to `title` |
| `galette_download_url` | no | Base URL `plugin.archive` builds on |

</div>

The whole cartouche disappears when a site declares no download at all.

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

Pages sharing a `ref` are offered to each other in the language selector, and
the menu keeps the reader inside their language. Declare one `defaults`
entry per language you publish:

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

Interface strings — the menu labels, the maintainer sentences, the cartouche, the
footer — live in `_includes/i18n.html`, with the language names in
`_includes/lang-name.html`. They are Liquid rather than `_data/i18n.yml` because
Jekyll themes only share `_layouts`, `_includes`, `_sass` and `assets`; a
`_data` file would be invisible to every site using the theme.

Currently shipped: English, français, Deutsch, español, italiano, português,
português do Brasil, slovenščina, українська, தமிழ். Anything else falls back
to English.

The strings the menu and the cartouche need are part of that set, so adding a
language means one `when` branch in each of the two includes.

The eight strings the menu and the cartouche need are part of that set, so a new
language means one `when` branch in each of the two includes.

## Licence

The theme is GPL-3.0-or-later, like the galette.eu stylesheets it derives from.
Site contents are expected to be CC BY-SA 4.0, as stated in the footer.
