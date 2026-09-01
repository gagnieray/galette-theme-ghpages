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

### Rebuilding the sites

A site using `remote_theme` downloads this repository when Pages builds it, so it
keeps serving the previous version of the theme until Pages builds it again —
nothing triggers that on its own. `bin/rebuild-consumers`, run by
`.github/workflows/rebuild-consumers.yml`, asks for one.

The sites are **discovered, not listed**: every non-archived repository of the
organisation that has a Pages site whose `_config.yml` references
`galette/theme-ghpages`. A hand-kept list went stale twice over — the day
`plugin-fullcard` moved between organisations, and the day stripe, legalnotices
and helloasso adopted the theme on a `gh-pages-galette-theme` branch of their
own. Discovery also means a repository that does not use the theme is never
rebuilt for nothing.

The scan follows the Pages source rather than assuming it: the branch can be
anything, and a source under a subdirectory has its `_config.yml` there. The
match is on the theme slug, not a whole line, so pinning to a tag
(`galette/theme-ghpages@v1`) still counts.

```bash
DRY_RUN=1 ./bin/rebuild-consumers      # list the sites without rebuilding
THEME_ORG=… THEME_SLUG=… EXTRA_REPOS=… # scan elsewhere, or add repos outside the org
```

The workflow exposes the same dry run through *Run workflow*.

It runs when `_layouts/`, `_includes/`, `_sass/` or `assets/` change on `main`,
and also after *Regenerate i18n* completes. That second trigger is not
redundant: the regeneration commits `_includes/` with `GITHUB_TOKEN`, and such a
push starts no workflow, so a translation update would otherwise never reach the
sites.

**It needs a token, and this is why.** `GITHUB_TOKEN` is minted for the workflow's
own repository and cannot act on another one, whatever permissions the workflow
declares — so it can never rebuild a plugin site. The workflow reads
`secrets.CONSUMER_PAGES_TOKEN` instead, which needs the **Pages** repository
permission at *write* on the consuming repositories. An organisation secret
shared with this repository covers them all at once. A fine-grained token limited
to Pages is enough; it needs no code access.

Without the secret the workflow does not fail, it warns and rebuilds nothing —
so a fork or a contributor is not blocked. When a rebuild is refused it does
fail, because a site silently serving a stale theme is the thing this exists to
prevent.

## 2. A plugin site's pages

One component per page, because a component maps one file mask. Rather than
creating them by hand, create one and let the **Component discovery** add-on
create the rest — it also picks up a page added later.

### Create the first component

*Add new translation component*, **From version control**:

| Setting | Value |
| --- | --- |
| Component name | `Fullcard site: documentation` |
| Repository | `https://github.com/galette-plugins/plugin-fullcard.git` |
| **Branch** | **`gh-pages`** — not the default branch |
| File format | **Markdown file** |
| File mask | `*/documentation.md` |
| Monolingual base language file | `documentation.md` |
| Template for new translations | `documentation.md` |
| Source language | English |
| License | GPL-3.0-only |

Then, in *Manage → Settings*, under **File format parameters**:

* **Translate front matter values** — on. The pages carry `title` and
  `description` in their front matter, and the tagline in the header comes from
  `description`. Without this the front matter is offered as one opaque block.
* **Deduplicate identical strings** — on. It keeps translations stable when a
  table row or a section moves.

### Add the discovery add-on

*Manage → Add-ons → Component discovery* (`weblate.discovery.discovery`):

| Field | Value |
| --- | --- |
| Regular expression | see below |
| File format | Markdown file |
| Customize the component name | `Fullcard site: {{ component }}` |
| Define the monolingual base filename | `{{ component }}.md` |
| Define the base file for new translations | `{{ component }}.md` |
| Clone add-ons from the main component | on |
| Remove components for inexistent files | off, at least to start with |

```
(?P<language>nb_NO|pt_BR|ota|ar|br|ca|de|es|fr|it|oc|pt|ru|si|sl|ta|tr|uk)/(?P<component>[^/]+)\.md
```

**Loose on the page name, exact on the language**, and that asymmetry is the
point. `[^/]+` picks up a page added later — an `installation.md` needs no change
here. The language group cannot be loose in the same way: `[a-z]{2,3}` would take
`doc/`, `img/`, `api/` and `css/` for languages and have Weblate create them, and
those are ordinary directory names on a documentation site.

Because the nineteen languages are a project constant, this expression is the
same for every Galette plugin site — copy it as is. `en` is absent on purpose: it
is the base file, at the root. Regenerate it from the theme's language table when
Galette gains a language:

```bash
ruby -ryaml -e 'l = YAML.load_file("i18n/languages.yml").keys - ["en"]
  puts "(?P<language>#{l.sort_by { |x| [-x.length, x] }.join("|")})/(?P<component>[^/]+)\\.md"'
```

Weblate derives the file mask by replacing the language group with `*`, giving
`*/index.md` and `*/documentation.md` — and `*/installation.md` the day such a
page appears. The add-on runs on installation and after every repository update,
so a page or a language added later needs nothing.

**File format parameters are not part of the add-on's configuration.** Cloning
add-ons does not clone them, so check *Translate front matter values* and
*Deduplicate identical strings* on each component the add-on creates.

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

The order this forces, when component discovery creates the components:

1. **Create the discovery component.** It finds `*/index.md` and
   `*/documentation.md` and creates one component per page — you do not create
   them by hand, so there is no "upload first" option.
2. **Expect Weblate to flatten the translated files.** Considering those
   languages untranslated, its first write replaces each `<lang>/*.md` with its
   own output, most likely through a pull request. This is not a loss: the
   content stays in the branch history, so
   `git show <commit>:de/documentation.md` gets any of them back.
3. **Upload the translations, one file per language per component.** In each
   component, per language, *Files → Upload translation*. An explicit upload goes
   through the parser and does populate Weblate, unlike a repository change.
4. **Weblate then opens a pull request** re-adding the files it now considers
   translated.

**Keep `doc-plugins-fullcard` until step 3 is done.** Its strings feed the project
translation memory, so a translator is offered the old wording as a full match
instead of retyping it — and it is the safety net if an upload goes wrong.

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
