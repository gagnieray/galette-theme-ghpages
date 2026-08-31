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
maintained by the Galette team or by a community member.

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
    values: {layout: default, lang: en}
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
| `maintainer` | yes | `core` shows the orange “Maintained by the Galette team” pill, `community` the grey one. Anything else is treated as `community`. |
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
| `plugin.release_url` | no | Overrides the derived stable URL; falls back to the repository's releases page |
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

A page's front matter carries only a `ref` — stable across languages — and a
`title`:

```yaml
---
ref: doc          # home, doc, …
title: Documentation
---
```

The language comes from the file's directory through path-scoped `defaults`,
never from the front matter. That is deliberate: it lets Weblate write
translated Markdown without having to produce a correct `lang` key.

```
index.md               -> /                      lang: en   ref: home
documentation.md       -> /documentation.html     lang: en   ref: doc
fr/index.md            -> /fr/                    lang: fr   ref: home
fr/documentation.md    -> /fr/documentation.html  lang: fr   ref: doc
```

Pages sharing a `ref` appear in each other's language selector, and the menu
keeps the reader inside their language, falling back to `default_lang` when a
page has no translation. Add one `defaults` entry per published language, and
never set `permalink` on a translated page — the URL comes from the path, so two
languages cannot collide.

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

Interface strings live in `_includes/i18n.html`, and language names in
`_includes/lang-name.html` — as Liquid rather than `_data/i18n.yml`, because
Jekyll themes only share `_layouts`, `_includes`, `_sass` and `assets`; a
`_data` file would be invisible to the sites using the theme. Adding a language
means adding a `when` branch to both files.

Shipped: English, français, Deutsch, español, italiano, português, português do
Brasil, slovenščina, українська, தமிழ். Anything else falls back to English.

## Licence

Theme code and stylesheets: **GPL-3.0-or-later**, like the galette.eu
stylesheets they derive from (see `LICENSE`).
Site contents: **CC BY-SA 4.0** (see `LICENSE.contents`), which is what the
footer states.

PT Sans is under the SIL Open Font License, see `assets/fonts/OFL.txt`.
