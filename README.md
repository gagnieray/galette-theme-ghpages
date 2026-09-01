# The Galette theme

[![CI](https://github.com/galette/theme-ghpages/actions/workflows/ci.yml/badge.svg)](https://github.com/galette/theme-ghpages/actions/workflows/ci.yml)
[![License](https://img.shields.io/github/license/galette/theme-ghpages)](https://github.com/galette/theme-ghpages/blob/main/LICENSE)

*The GitHub Pages theme for [Galette](https://galette.eu) plugin websites. You can
[preview the theme to see what it looks like](https://galette.github.io/theme-ghpages/),
or [use it today](#usage).*

![Thumbnail of the Galette theme](thumbnail.png)

It carries the same identity as galette.eu — PT Sans, the orange `#ffb619`, the
blue `#007baa`, the grey band behind the logo, the sidebar menu and the orange
download cartouche — and it states on every page whether the plugin is
maintained by the Galette team or by a community member, with a pill whose icon
and outline carry the distinction.

## Usage

On your plugin's `gh-pages` branch, in `_config.yml`:

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
    values: {layout: default}
```

Then set GitHub Pages to build from the `gh-pages` branch, root directory.
Nothing else is needed — no workflow, and no `_layouts` of your own (a local
`_layouts/default.html` would shadow the theme's).

The theme repository **must stay public**: `jekyll-remote-theme` only accepts
"a public GitHub-hosted Jekyll theme", and every consuming site's build would
break the moment it went private.

### Configuration variables

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

### The download cartouche

The orange box in the header points at wherever the plugin's archives actually
live — releases on GitHub, nightly builds on galette.eu:

| Key | Required | What it does |
| --- | --- | --- |
| `plugin.archive` | no | Archive base name. With `plugin.version` it builds both URLs on galette.eu: `{archive}-{version}.tar.bz2` and `{archive}-dev.tar.bz2`. |
| `plugin.version` | no | Version shown on the stable button, `latest` when absent |
| `plugin.min_galette` | no | Minimum Galette version, shown beside the maintainer pill |
| `plugin.release_url` | no | Overrides the derived stable URL; falls back to the repository's `releases/latest` |
| `plugin.nightly_url` | no | Overrides the derived nightly URL |
| `plugin.name` | no | Label on both buttons, defaults to `title` |
| `galette_download_url` | no | Base the two URLs are built on, defaults to `https://galette.eu/download/plugins` |

The whole cartouche disappears when a site declares no download at all.

Galette plugin releases live on galette.eu rather than on GitHub — plugin-fullcard
has no GitHub release at all — so declaring `archive` and `version` is the normal
way, and the version then lives in exactly one place.

The version and the minimum Galette version belong here and nowhere else: a
number written into a page becomes a string a translator has to carry, in every
language, and has to be bumped in each of them at every release.


### The menu

The menu lives in the sidebar, exactly as on galette.eu, and folds into an
off-canvas panel on small screens — driven by `:target`, so it needs no
JavaScript. It lists the site's own pages first (Home, Documentation), then the
bug tracker and the source repository, then a Galette block.

### Pages and languages

A page's front matter carries only what a reader sees:

```yaml
---
title: Documentation
description: Full member card as PDF
---
```

Everything structural is derived. **The language is the first path segment**, when
that segment is one of the languages in `i18n/languages.yml`; **translations of a
page are the pages sharing its file name**.

```
index.md               -> /                      en
documentation.md       -> /documentation.html     en
fr/index.md            -> /fr/                    fr, paired with index.md
fr/documentation.md    -> /fr/documentation.html  fr, paired with documentation.md
```

So a site needs no `defaults` beyond the layout, and a translation Weblate adds
appears on its own. Do **not** put `lang` in `defaults`: it would override the
derivation for every page, translated ones included. And never set `permalink` on
a translated page — the URL is what carries the language.

This matters for Weblate: with *Translate front matter values* enabled, any key
in the front matter is offered for translation. An identifier there would be
handed to translators and flagged on every language by the format's strict-same
check, so there is none. `title` and `description` are both legitimately
translatable — `description` is the tagline in the header.

The menu still needs to know which page is the home and which is the
documentation: `index.md` and `documentation.md` by name, or an explicit `ref` of
`home` or `doc` for a site that names them otherwise. Any other page is paired
across languages all the same.

The picker is a `<details>` disclosure: it shows the current language and folds
the others away, which a flat list of nineteen could not do. Being native HTML
it opens with the keyboard and works with JavaScript disabled; the theme's script
only adds dismissing it with Escape or a click elsewhere. It appears only when
the page actually has a translation.

### Wide content

Wrap wide tables in `<div class="table-wrapper" markdown="1"> … </div>` so they
scroll inside themselves instead of making the page scroll sideways. The theme
makes such a container — and any scrolling code block — keyboard-reachable on
its own; you do not need to add `tabindex`.

## Development

The demo site at the repository root is what GitHub Pages publishes, and it
exercises every layout, include and stylesheet the theme ships — so CI building
it is a real test.

```bash
bundle install
bundle exec jekyll build      # script/server to preview
```

Local preview reproduces the GitHub Pages toolchain through the `github-pages`
gem (Jekyll 3.9, jekyll-sass-converter 1.5, Ruby Sass 3). That toolchain is why
everything under `_sass/` uses `@import` and not `@use`: `@use` builds fine on a
modern Sass and then fails on GitHub Pages.

Asset URLs in the SCSS are relative to the compiled stylesheet
(`../images/bg.png`, not `/site/assets/images/bg.png` as on galette.eu), so the
theme works under any `baseurl`.

### Previewing a plugin site against the theme

`remote_theme` always downloads the theme's **pushed** default branch, and caches
it under `/tmp/jekyll-remote-theme-*`. A theme change is therefore invisible to a
consuming site until it is pushed here — which is the usual reason a site looks
like it ignored an edit.

To try a site against an unpushed theme, drop `remote_theme` from its config and
link the four directories a theme shares:

```bash
ln -s /path/to/theme-ghpages/{_layouts,_includes,_sass,assets} .
bundle exec jekyll build --config _config.yml,_config.notheme.yml
```

Remove the symlinks before committing — a local `_layouts/default.html` shadows
the theme's, which is exactly what makes this work and also what would silently
freeze the site on an old layout if left behind.

### Languages

The theme knows the nineteen languages Galette translates into — the list shared
with the core, the manual and every plugin, at
<https://hosted.weblate.org/projects/galette/>. `i18n/languages.yml` holds their
codes, their autonyms (computed the way `Galette\Core\I18n` computes them) and
which are right-to-left.

Interface strings live in `i18n/strings/<lang>.yml`, one file per language, which is what
Weblate translates. `bin/build-i18n` turns them into `_includes/i18n.html` and
`_includes/lang-name.html`, and both are committed: Jekyll themes only share
`_layouts`, `_includes`, `_sass` and `assets`, so a `_data/i18n.yml` would be
invisible to the sites using the theme, and Liquid is no format to hand a
translator. CI fails if the two drift.

```bash
./bin/build-i18n            # regenerate after editing i18n/strings/*.yml
./bin/build-i18n --check    # what CI runs
```

`i18n/languages.yml` is reference data and stays out of the translation file
mask.

You rarely need to run this by hand: `.github/workflows/i18n.yml` regenerates and
commits the includes whenever `i18n/strings/**`, `i18n/languages.yml` or
`bin/build-i18n` changes on `main`, which is what keeps Weblate's translations
flowing through without an add-on.

Sites built on the theme are then asked to rebuild, since `remote_theme` is
resolved at their build time and nothing else would tell them the theme moved.
They are discovered by scanning the organisation, not kept in a list. Both are
described in [WEBLATE.md](WEBLATE.md), including the token the cross-repository
rebuild needs.

Ten languages have their own strings; the nine others render the English ones
until they are translated, while still declaring their own `lang` and text
direction. A language absent from `languages.yml` falls back to English
entirely.

Right-to-left languages get `dir="rtl"` on `<html>`, Galette's mirrored header
photo, and a layout built on logical properties, so nothing needs a second
stylesheet.

## Licence

Theme code and stylesheets: **GPL-3.0-or-later**, like the galette.eu
stylesheets they derive from (see `LICENSE`).
Site contents: **CC BY-SA 4.0** (see `LICENSE.contents`), which is what the
footer states.

PT Sans is under the SIL Open Font License, see `assets/fonts/OFL.txt`.
