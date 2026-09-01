# Setting up Weblate for a Galette plugin site

Two kinds of component are involved: this theme's interface strings, and the page
content of each plugin site. They use different file formats and behave
differently, so they are described separately.

Settings below mirror the existing Galette components (checked against
`galette/plugin-oauth2`), so a new component sits in the project the same way the
others do.

## Prerequisites

* The Weblate **GitHub App must be installed on the repository**, and granted
  write access. Every Galette component uses `vcs: github-app` with an empty
  push URL — Weblate pushes through the app and opens pull requests, so no
  deploy key is needed. A component created before the app is installed will
  fail on its first push, not on creation.
* For a plugin site the branch is **`gh-pages`**, not the default branch. Getting
  this wrong is silent: Weblate happily translates files from `develop` that the
  site never serves.
* Every write Weblate makes to `gh-pages` triggers a Pages rebuild. That is
  wanted, but it means `push_on_commit` and `commit_pending_age` decide how often
  the site rebuilds. The Galette components use `push_on_commit: true` and
  `commit_pending_age: 24`.

## 1. The theme's interface strings

Menu labels, the maintainer sentences, the download cartouche, the footer — about
twenty strings shared by every plugin site.

| Setting | Value |
| --- | --- |
| Repository | `https://github.com/galette/theme-ghpages.git` |
| Branch | `main` |
| File format | **YAML file** (API `yaml`) |
| File mask | `i18n/strings/*.yml` |
| Monolingual base language file | `i18n/strings/en.yml` |
| Template for new translations | `i18n/strings/en.yml` |
| Source language | English |
| License | GPL-3.0-only |
| Merge style | Rebase |

Ten languages already have a catalogue and will be imported as translated; the
nine others appear empty and fall back to English on the sites until filled.

**`i18n/languages.yml` is deliberately outside the mask.** It is reference data —
language codes, autonyms, text direction — not translatable content. A mask of
`i18n/*.yml` would make Weblate treat `languages` as a language code.

### Keeping the generated Liquid in step

The Liquid the theme ships is generated from these YAML files, so `bin/build-i18n`
has to run once translations land. The *Execute script* add-on is not available
on hosted.weblate.org, so `.github/workflows/i18n.yml` does it instead: on a push
to `main` touching `i18n/strings/**`, `i18n/languages.yml` or `bin/build-i18n`, it
regenerates the two includes and commits them if they changed. Nothing runs for
an ordinary commit.

Two things make it safe against looping: its own commit touches only
`_includes/`, which no path filter matches, and a push authenticated with
`GITHUB_TOKEN` does not start workflows.

The CI drift check therefore **warns rather than fails**. A Weblate pull request
only ever touches `i18n/strings/*.yml`, so the includes are stale until the merge
— expected, and not something to block on. On a human pull request the same
warning is the reminder to run the script.

**If `main` is protected**, the workflow's push is refused. Either allow
`github-actions[bot]` to bypass the restriction, or give the workflow a token
that can. There is no way around it with the default `GITHUB_TOKEN`.

A consuming site still needs its own Pages rebuild to pick up new strings — a
theme change does not trigger one. That is a `POST /repos/{owner}/{repo}/pages/builds`
per site, and could be another workflow step once the list of consuming sites is
settled.

## 2. A plugin site's pages

One component per page, because a component maps one file mask.

| Setting | `…-site-home` | `…-site-doc` |
| --- | --- | --- |
| Repository | the plugin repository | same |
| Branch | `gh-pages` | `gh-pages` |
| File format | **Markdown file** (API `markdown`) | same |
| File mask | `*/index.md` | `*/documentation.md` |
| Monolingual base language file | `index.md` | `documentation.md` |
| Template for new translations | `index.md` | `documentation.md` |

File format parameters, both components:

* **Translate front matter values** — on. The pages carry `title` and
  `description` in their front matter, and the tagline in the header comes from
  `description`. Without this the front matter is offered as one opaque block.
* **Deduplicate identical strings** (`markdown_merge_duplicates`) — on. It keeps
  translations stable when a table row or a section moves.

`ref` is in the front matter too and must not be translated: it is what pairs a
page with its other languages. Mark it read-only, or tell translators to leave it
alone.

### The one trap that matters

The Markdown format is **monolingual, and Weblate does not read translations back
from the repository**. The documentation is explicit:

> Unlike most other formats, the changes in the translation files will not be
> imported to Weblate because it can not be done reliably. The source of truth
> for the translations is Weblate not the translated file.

So a translated `de/documentation.md` sitting in the branch counts for nothing:
Weblate will consider German untranslated and, on its first write, replace the
file with its own output. For `plugin-fullcard` that would discard the nine
catalogues recovered from the Sphinx manual.

Two ways to avoid losing them, and they combine:

1. **Upload each translation once, after creating the component.** In the
   component, per language, *Files → Upload translation*, with the existing
   `<lang>/documentation.md`. An explicit upload goes through the parser and does
   populate Weblate, unlike a repository change.
2. **Do not remove `doc-plugins-fullcard` until the new components are
   populated.** Its strings feed the project translation memory, so translators
   are offered the old wording as a full match instead of retyping it.

Also worth knowing: Weblate labels this format's support as *under development*,
with behaviour that may change between releases. Worth a check after a Weblate
upgrade.

## 3. Adding a language

Nothing to do in the theme: all nineteen languages Galette translates into are
already declared in `i18n/languages.yml`, and a language with no catalogue simply
renders the English strings while keeping its own `lang` attribute and text
direction.

On the plugin site, add its `defaults` entry in `_config.yml` — `plugin-fullcard`
already lists all nineteen — and Weblate can then create the page files itself.
