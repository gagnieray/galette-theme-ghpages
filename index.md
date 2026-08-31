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
  min_galette: "1.3.0"
defaults:
  - scope: {path: ""}
    values: {layout: default, lang: en}
```

Then make sure GitHub Pages is set to build from the `gh-pages` branch, root
directory. Nothing else is needed: no workflow, no `_layouts` of your own.

<div class="table-wrapper" markdown="1">

| Key | Required | What it does |
| --- | --- | --- |
| `title` | yes | Plugin name, shown as the page heading |
| `description` | yes | Tagline under the heading, and the meta description. A page may override it with its own `description` front matter — that is how a translated page gets a translated tagline. |
| `maintainer` | yes | `core` shows the orange “Maintained by the Galette team” pill, `community` the grey one. Anything else is treated as `community`. |
| `plugin.archive` | no | Archive base name, used to build the nightly download link |
| `plugin.min_galette` | no | Minimum Galette version the plugin needs |
| `author` | no | Name in the copyright line, defaults to Johan Cwiklinski |
| `default_lang` | no | Language the navigation falls back to when a page has no translation, defaults to `en` |
| `galette_contact_url` | no | Overrides the “Get in touch” target |

</div>

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
the navigation keeps the reader inside their language. Declare one `defaults`
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

Interface strings — the four navigation labels, the maintainer sentences, the
footer — live in `_includes/i18n.html`, with the language names in
`_includes/lang-name.html`. They are Liquid rather than `_data/i18n.yml` because
Jekyll themes only share `_layouts`, `_includes`, `_sass` and `assets`; a
`_data` file would be invisible to every site using the theme.

Currently shipped: English, français, Deutsch, español, italiano, português,
português do Brasil, slovenščina, українська, தமிழ். Anything else falls back
to English.

## Licence

The theme is GPL-3.0-or-later, like the galette.eu stylesheets it derives from.
Site contents are expected to be CC BY-SA 4.0, as stated in the footer.
