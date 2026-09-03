# Migrating a Galette plugin to this theme

Six core plugins are still in the `galette` organisation and have no website:
**auto, paypal, maps, events, objectslend, activities**. This is the procedure to
move one of them, written from what the first two — `plugin-fullcard` and
`plugin-oauth2` — cost to get right.

`auto` is the worked example throughout. Section 5 carries what differs for the
other five.

Weblate component settings are in [WEBLATE.md](WEBLATE.md); this file only says
when to create them and gives the per-plugin values.

## 1. What already happens on its own

Nothing below needs configuring per plugin. Checked in production on 2026-09-02:
the theme was pushed at 05:21:43 and the three consuming sites were rebuilt at
05:22:0x.

* **A site is discovered, not registered.** `bin/rebuild-consumers` scans the
  organisation for Pages sites whose `_config.yml` references this theme, so a
  new one is picked up as soon as it exists.
* **Theme changes reach the sites.** A push touching `_layouts`, `_includes`,
  `_sass` or `assets` rebuilds every consuming site, and so does a translation
  landing on the theme's own strings.
* **A release refreshes the version.** `galette/.github/actions/release-plugin`
  requests a Pages build after a tag, so the download cartouche stops naming the
  previous release without anyone asking.
* **Languages are already known.** The theme carries the nineteen languages
  Galette translates into, with their native names and text direction, and takes
  a page's language from its directory. A language Weblate adds on a translator's
  request needs no change in the plugin repository.

## 2. The traps already paid for

Every line here cost a round trip on fullcard, oauth2 or auto.

| # | Trap | Where it bites |
|---|---|---|
| 1 | GitHub issues are **disabled** on five of the six, so `tracker_url` is required and `/issues` must not be linked — but they are **enabled on activities**, which is where its bugs go | check `has_issues` per repository |
| 2 | Adding front matter to `README.md` turns it into `README.html` and the site **loses its root** — GitHub Pages only renders it as the index while it has none | oauth2, reverted |
| 3 | A version written into a page becomes a string every translator carries and re-bumps at each release | the two shields badges in each `.rst` |
| 4 | A catalogue may have translated a **real path** — Tamil had translated the `plugins` directory name | review every generated translation |
| 5 | `lang: en` in `defaults` overrides the derived language on **every** page, translated ones included | theme, fullcard, oauth2 |
| 6 | `fr` and `fr_FR` are duplicate locale directories in the documentation repository | publish `fr` only |
| 7 | An empty translation licence on the parent Weblate component makes discovery fail on the children | `{'license': ['Ce champ ne peut pas être vide.']}` |
| 8 | **File format parameters set after Weblate's first write.** Until *Translate front matter values* is on, the front matter is not a unit, so Weblate copies the source one — destroying every translated `title`. Enabling it afterwards re-reads the now-English file and records English as the translation. | fullcard lost eight titles this way |
| 9 | Seeding English text into a language file makes Weblate record it as that language's translation, and the component reports 100 % | fullcard, seven languages at 100 % with twelve English units |
| 10 | A file mask and a base file that disagree give "files found several times" | a parent on `*/index.md` whose base was still `documentation.md` |
| 11 | `vcs: git` means "Outbound translation delivery is manual" — nothing ever reaches GitHub | use the **GitHub App** integration |
| 12 | The Markdown format **does not read translations back** from the repository | upload them once, after the components exist |
| 13 | The Pages source branch is not always `gh-pages` | stripe, legalnotices and helloasso publish from `gh-pages-galette-theme` |
| 14 | `bin/release` predates the Actions pipeline on **every plugin still to migrate** — the workflows alone are not enough | auto's first Nightly died on `import urlgrabber.progress` |
| 15 | The three CI conditions can each hold a **different** stale value, and the branch they name can be wrong too | oauth2 had `galette-oauth2` twice, `plugin-oauth2` once, and `master` where its branch is `main` |
| 16 | Deleting an immutable release **burns its tag for good** — GitHub then refuses to create that ref at all | oauth2 cannot use the `nightly` tag and rolls on `dev` instead |

## 3. The procedure

### Step 1 — the organisation in the CI condition

`.github/workflows/ci-linux.yml` compares `github.repository` in **three** places:
two `all-jobs` inputs and the `unit-tests` job name. All three name
`galette/plugin-auto` today and become `galette-plugins/plugin-auto` **at the
transfer, not before** — setting it early only makes the condition false in a
different way, and the full matrix stops expanding.

The checked-out path `plugin-auto` appears five more times in the same file and
does **not** change: the repository keeps its name, only its owner moves.

Read the three of them rather than replacing a pattern, because they do not
always agree — oauth2 held two different repository names, and named `master`
while its stable branch is `main`. Both branch names appear in the condition, so
check them against the repository too:

```sh
grep -o "github\.repository [!=]= '[^']*'" .github/workflows/ci-linux.yml | sort | uniq -c
grep -o 'refs/heads/[a-z]*' .github/workflows/ci-linux.yml | sort -u
git ls-remote --heads origin master main
```

Measured on the five left: all three occurrences already say `plugin-<name>`, and
the branches already match — `master` everywhere, `main` for activities. Only the
organisation changes.

### Step 2 — transfer the repository

To `galette-plugins`. GitHub redirects the old URLs, but `site.github.*` resolves
the new organisation, so transfer **before** creating `gh-pages` — otherwise the
menu and footer links point at `galette/…` while the pages point at
`galette-plugins/…`, and the site needs rebuilding to agree with itself.

### Step 3 — release workflows, and `bin/release`

Copy `.github/workflows/release.yml` and `nightly.yml` from `plugin-fullcard`.
The only edit is `nightly.yml`'s guard:

```yaml
if: github.repository == 'galette-plugins/plugin-auto'
```

Both delegate to `galette/.github/actions/release-plugin`, which builds with the
plugin's own `bin/release`, publishes the release, keeps one rolling `nightly`
prerelease, and requests the Pages build.

**And that script needs the same treatment Galette's own got in `9848314b8`.**
Copying the workflows is not enough: only fullcard and oauth2 carry those
changes, the nine other plugins carry none of them, and each missing piece is a
failed run rather than a degraded one.

| Change | Why the runner needs it |
|---|---|
| drop `urlgrabber.progress` from the imports | nothing in the script uses it, and the action installs only GitPython and termcolor — this is an immediate `ModuleNotFoundError` |
| add `--no-sign` | the release path is invoked with it, to skip a signature no runner can produce; argparse rejects the whole command without it |
| `os.makedirs(local_dl_repo)` | the archive is built in `<repo>/dist`, which a fresh checkout does not have |
| `if tag is None: continue` in the two loops that read `tag.tag` | a lightweight tag has no tag object; the two migrated plugins have none, but nothing prevents one |
| the `todrop` list, plus `.github`, `bin` and `tests` | otherwise the published archive ships `phpstan.neon`, `.scrutinizer.yml`, the test suite and the workflows |

`plugin-fullcard`'s script is the reference: once ported, a plugin's `bin/release`
differs from it only by the copyright year and the plugin name. Diff them before
dispatching anything.

Two things in `release.yml` look wrong and are not, so do not "fix" them back:
it checks out the **default branch** rather than the tag — `bin/release` archives
the tag's tree with `git archive <tag>`, but the tooling doing it has to be the
current one — and it asks for `pages: write`, because the plugin page reads the
latest release and has to be rebuilt after one.

The shared action has four inputs worth knowing, all with usable defaults:

| Input | When it is needed | State of the five |
|---|---|---|
| `nightly-tag` | the default `nightly` tag is unusable, typically burnt by a deleted immutable release | not needed: none of them has a `nightly` or `dev` tag |
| `nightly-branch` | the plugin's development branch is not `develop`, which `bin/release -n` reads | not needed: all six default to `develop` |
| `php-version`, `php-extensions` | the plugin has a `composer.json` whose install needs a specific runtime | not needed: none of the five has one — only oauth2 does |
| `dry-run` | build and upload an artifact, publish nothing | see the next step, the Nightly is a better rehearsal |


Prove it locally rather than through a run — the action's own comment says
`bin/release` ignores every subprocess exit code, so a broken build can still
exit 0:

```sh
python3 bin/release -V -n -l -d /tmp/out     # nightly path, exactly as the action calls it
python3 bin/release -V -Y --no-sign -l -d /tmp/out -v 99.99.99   # release path, stops on the unknown version
```

The first must leave exactly one `.tar.bz2` in `/tmp/out` — the action fails on
any other count — carrying `<prefix>/_define.php` and none of the development
files. `-d` is the **download URL**, not a destination: in local mode the upload
becomes a copy into it, which is what keeps the build off `ssh.galette.eu`.
Remove the `dist/` it leaves behind, it is not ignored.

### Step 4 — cut a first release

The page reads `releases/latest`; with no release at all the cartouche falls back
to the releases page with the label *latest*. **Only auto has GitHub releases —
2.2.0 and 2.2.1, published by hand — the five others have none.**

Every plugin's `_define.php` matches its newest tag — `2.2.1` for auto, paypal,
maps, events and objectslend, `1.1.1` for activities — so a `workflow_dispatch`
of *Release* is enough.

A release built by hand proves nothing about the workflow: it runs on a machine
where `urlgrabber` happens to be installed, which is exactly how auto's missing
dependency stayed invisible until the first Nightly.

**Publish through the workflow, not by hand.** `Release` takes a tag and a
`dry-run` flag that defaults to **true**, so a first dispatch builds the archive
and publishes nothing — the cheapest possible rehearsal of the release path,
including the `--no-sign` flag. Then dispatch it for real, **oldest version
first**, so the newest ends up flagged Latest:

```sh
gh workflow run Release --repo galette-plugins/plugin-objectslend --ref develop \
  -f version=2.2.1 -f dry-run=true      # rehearsal
gh workflow run Release --repo galette-plugins/plugin-objectslend --ref develop \
  -f version=2.2.0 -f dry-run=false     # then 2.2.1
```

Done that way the archive carries a build attestation, and the action asks for
the Pages build itself: objectslend's cartouche went from 2.2.0 to 2.2.1 with no
intervention.

**A release published by hand does not rebuild the site.** Only the action asks
for a Pages build, so a release created from the tarballs on galette.eu leaves
the cartouche on its `releases/latest` fallback until the next build. Ask for one:

```sh
gh api -X POST repos/galette-plugins/plugin-maps/pages/builds
```

That is how maps came to advertise its 2.2.1 archive. And until a Nightly has
run, the nightly button points at an asset that does not exist — a 404 the
cartouche cannot know about, since it derives the URL from `archive`.

**So dispatch *Nightly* first.** It runs the same action and the same script, and
publishes into a rolling prerelease instead of a version — a failure costs
nothing and shows up in about twenty seconds. Auto's second attempt produced
exactly what the cartouche expects: a prerelease on the `nightly` tag carrying
one asset, `galette-plugin-auto-dev.tar.bz2`. Dispatch *Release* only once that
has passed.

Check that pairing before dispatching, though. A version declared without a tag
gives a page announcing a release nobody can download, which is the state
fullcard was in before its first Actions release.

### Step 4b — the older releases, later

Once a plugin is on the new organisation, its whole history can be republished as
GitHub releases. The tarballs are already served:

* recent versions: <https://galette.eu/download/plugins/>
* everything before that: <https://galette.eu/download/archives/plugins/>

Both are plain indexes of `galette-plugin-<name>-<version>.tar.bz2`, each with a
detached `.asc` or `.sig` beside it — the signature GitHub's build attestation
replaces for anything built by the action from now on.

Three things to reconcile rather than assume, checked on events:

* **the two sets differ.** It has nineteen annotated tags and eighteen published
  archives: `1.3.1` was tagged and never published. A backfill has to decide what
  to do with a tag that has no tarball, and with a tarball whose tag is missing.
* **every archive is listed twice** in those indexes, so deduplicate before
  counting — 240 links for 120 files.
* **mark them explicitly as not the latest** (`gh release create --latest=false`),
  or publishing an old version demotes the current one on the repository page and
  in the theme's download box, which reads `releases/latest`.

Rebuilding those archives from their tags is not the point — the published bytes
are what users already have. Attach the file as it stands.

### Step 5 — the orphan `gh-pages` branch

```
_config.yml
index.md                          showcase, from the README
documentation.md                  from source/plugins/auto.rst
<lang>/index.md                   one directory per published language
<lang>/documentation.md
images/                           screenshots, when the .rst has any
.gitignore   Gemfile   README.md  the last two excluded from the build
```

Create it with a worktree so the plugin checkout keeps serving your local
Galette:

```bash
git worktree add --orphan -b gh-pages ~/Workdir/plugin-auto-ghpages
```

`_config.yml` for `auto`:

```yaml
remote_theme: galette/theme-ghpages

title: Galette Auto
description: Plugin to manage Automobile clubs
author: Johan Cwiklinski

# Developed and maintained by the Galette team, unlike most of the repositories
# in this organisation — hence the pill in the header.
maintainer: core

# No version: releases are built by GitHub Actions, so the theme reads the latest
# one and derives both download URLs from `archive`. Nothing to bump.
plugin:
  name: Auto
  archive: galette-plugin-auto
  min_galette: "1.3.0"

# GitHub issues are disabled on this repository; the tracker is the Redmine
# project, as for Galette itself.
tracker_url: https://bugs.galette.eu/projects/galette-plugin-auto

# The theme takes the language from the first path segment and the page identity
# from the file name, so a translation Weblate adds needs nothing here. No `lang`
# — it would override the derivation for every page.
defaults:
  - scope: {path: ""}
    values: {layout: default}

# Without this, README.md would be copied into the published site.
exclude:
  - README.md
  - Gemfile
  - Gemfile.lock
  - vendor
```

Two values are **read, never guessed**:

* `archive` comes from `bin/release` (the `archive_name` assignment, around line
  178). `legalnotices` uses `galette-plugin-legal-notices`, so the repository name
  is not a reliable source.
* `min_galette` is `compver` in `_define.php`.

A third one only when step 3 had to override `nightly-tag`: the cartouche builds
the nightly URL from the `nightly` tag, so a plugin rolling on another tag needs
`plugin.nightly_url` spelt out or its nightly link points at an asset that does
not exist. oauth2 is the only one so far.

And do **not** add a `_layouts/` of your own: a local `_layouts/default.html`
shadows the theme's, which is the one way to freeze a site on an old layout
without noticing.

### Step 6 — content and translations

The Sphinx page stays where it is; its content is **carried over**, not moved.
`source/changelogs/galette_07.rst` links `:doc:`plugin Auto </plugins/auto>``, so
deleting the page would break the changelog build.

* **Drop the two shields badges.** They hold the version in their URL, which
  would put it inside a translated string in every language. The theme's
  cartouche is the download UI.
* **Rewrite the cross-reference.** `:ref:`… <plugins_managment>`` points at a
  label in the manual; on the site it becomes a link to
  `https://doc.galette.eu/en/master/plugins/index.html#plugins-managment`.
* **Demote the headings one level.** The theme already renders the site title as
  `h1`.
* **Copy the screenshots** from
  `source/_styles/static/images/plugin-<name>/` into `images/`. `auto` has none.
* **Recover the translated paragraphs** from
  `source/locale/<lang>/LC_MESSAGES/plugins/auto.po`, converting the inline RST:
  ``` ``x`` ``` → `` `x` ``, ``` `text <url>`_ ``` → `[text](url)`, and the
  default role ``` `x` ``` → `` `x` `` since it only ever wraps technical
  literals here. **Read every generated file** — see trap 7.
* Front matter is **`title` and `description` only**. No identifier, no `lang`:
  with Weblate's *Translate front matter values* enabled, anything else there is
  handed to translators.

**Leave an untranslated paragraph out of the language file rather than seeding it
in English.** This is the one that bit on fullcard. Weblate imports whatever the
file contains and records it as the translation, so an English paragraph seeded
into `de/documentation.md` becomes "the German translation is this English text":
the component reports **100 % translated** while most of it is English, and no
translator will ever be shown those strings. On fullcard, twelve of fifteen units
ended up that way in seven languages — Weblate does flag them, ten or eleven
failing `strict-same` checks per language, but the progress bar says complete.

A file that only carries the paragraphs which were actually translated is
honest: Weblate reports it as partial, which is what it is, and offers the rest
for translation.

### Step 7 — enable Pages

Source: branch `gh-pages`, root `/`.

Enabling it leaves HTTPS unenforced, and that is not cosmetic: `http://` serves
the site in the clear with a 200, while an enforced site answers 301. oauth2
redirects, auto and fullcard do not.

**Tick *Enforce HTTPS* in Settings → Pages.** The API cannot do it on a freshly
enabled site — it answers `404 The certificate does not exist yet`, with or
without `source` in the body, and a missing permission would be a 403 — so the
checkbox is what creates that record. Check the result on the site itself rather
than the flag:

```sh
curl -sI http://galette-plugins.github.io/plugin-auto/ | head -1   # want a 301
```

### Step 8 — repository metadata, and the local remote

```bash
gh api -X PATCH repos/galette-plugins/plugin-auto \
  -f homepage='https://galette-plugins.github.io/plugin-auto/'
```

* `homepage` → the site URL, in **https** (fullcard and maps were on `http`). It
  404s until Pages is enabled, which is fine — it is the URL the site will have.
* `description` → the `desc` from `_define.php` reads well enough.
* Topics, including `galette-plugin`.

And in your own checkout, the remote still points at the old organisation. GitHub
redirects, so nothing breaks loudly — but `jekyll-github-metadata` reads that
remote, so a local preview of the site resolves `galette/…` and the menu and
footer links point at the wrong organisation until you either pass
`PAGES_REPO_NWO` or fix the remote:

```bash
git remote set-url origin git@github.com:galette-plugins/plugin-auto.git
```

Worth fixing while you are there: `events`'s description starts with a space,
`objectslend`'s does not mention Galette, `activities` has no topics at all, and
`maps` carries a `localistion-adherents` topic and a description saying
"Leaftleft".

### Step 9 — the documentation repository

In `source/plugins/index.rst`, replace the `:doc:` entry with the site:

```rst
* `Auto <https://galette-plugins.github.io/plugin-auto/>`_
```

Then **add `:orphan:` as the first line of `auto.rst`**. Since commit
`17c448e64` replaced the toctree with a bullet list, none of the seven plugin
pages is in any toctree and Sphinx warns once per page; `plugins-tiers.rst`
already sidesteps it that way.

The `.po` files and the `doc-plugins-*` Weblate components stay. While you are in
`index.rst`, `OjectsLend` is missing a *b*.

### Step 10 — Weblate

Two components per site, created from one plus the discovery add-on. Settings and
the order that the Markdown format forces are in [WEBLATE.md](WEBLATE.md). The
per-plugin values:

| Field | Value for `auto` |
|---|---|
| Component name | `Plugins websites: auto - documentation` |
| Repository | `https://github.com/galette-plugins/plugin-auto.git` |
| Branch | **`gh-pages`** |
| File mask / base | `*/documentation.md` / `documentation.md` |
| Translation licence | `CC-BY-SA-4.0` |
| Version control system | **GitHub App**, push URL empty |

Three of these block outright if you miss them: the branch (trap 13), the licence
(trap 7) and the VCS (trap 11).

**Set the file format parameters before Weblate writes anything.** *Translate
front matter values* and *Deduplicate identical strings* belong to the component
creation form, not to a second pass: until the first is on, the front matter is
not a translatable unit, so Weblate copies the source one over every language
file and every translated `title` is gone. Turning it on afterwards does not
recover them — it re-reads files that are already English and records English as
the translation, which then reads as done. That is how fullcard lost eight titles
(trap 8).

If a component is already in that state, the titles have to be re-entered by hand,
one string per language.

### Step 11 — verification

```bash
cd ~/Workdir/plugin-auto-ghpages
bundle install
PAGES_REPO_NWO=galette-plugins/plugin-auto bundle exec jekyll serve
```

Copy the `Gemfile` from fullcard's branch: it takes **Jekyll 4**, not the
`github-pages` gem, which pins liquid 4.0.3 — that calls `String#tainted?`,
removed in Ruby 3.2, so it cannot build on a current Ruby. `PAGES_REPO_NWO` is
only needed until the transfer, after which the `origin` remote answers.

On the published site:

* the language picker offers the languages you published, and each entry lands on
  that language's page;
* the menu is in the reader's language and its links stay inside it;
* the cartouche names the release you cut, and the nightly link resolves;
* the pill reads *Maintained by the Galette team*, and the required Galette
  version sits beside the title.

Then axe (`wcag2a`, `wcag2aa`, `wcag21a`, `wcag21aa`) at 360, 768 and 1280 px with
the picker both closed and open, and no page scrolling horizontally. And in the
documentation repository, `make html` followed by the same with `-D language=fr`,
with no new warning.

## 4. Per-plugin specifics

Values read from each plugin, not inferred.

| | archive | min_galette | DB | doc images | tracker | stable branch |
|---|---|---|---|---|---|---|
| auto | `galette-plugin-auto` | 1.3.0 | yes, `scripts/`, 10 tables | 0 | `galette-plugin-auto` | master |
| paypal | `galette-plugin-paypal` | 1.3.0 | yes + upgrade scripts | 3 | **`galette-plugin-paypa`** | master |
| maps | `galette-plugin-maps` | 1.3.0 | yes + upgrade scripts | 4 | `galette-plugin-maps` | master |
| events | `galette-plugin-events` | 1.3.0 | yes + upgrade scripts | 7 | **`evenements`** | master |
| objectslend | `galette-plugin-objectslend` | 1.3.0 | yes, **no pgsql upgrades** | 4 | `galette-plugin-objectslend` | master |
| activities | `galette-plugin-activities` | 1.3.0 | yes | 0 | **GitHub issues** | **main** |

Two tracker slugs are off convention and both answer 200: paypal's is truncated
(`galette-plugin-paypa`) and events' is French (`evenements`). `activities` has no
Redmine project at all, and needs none: it is the one repository with GitHub
issues enabled, so its `tracker_url` is
`https://github.com/galette-plugins/plugin-activities/issues`.

It was also the one shipping its GPL-3 text as `LICENSE` where its six siblings
ship `COPYING`, so the file every README badge and archive expects was missing.
Check that per plugin rather than assuming: a licence a tool cannot find is a
bug, not a detail.

**All five need database tables**, unlike fullcard. Their `documentation.md`
keeps its *Database initialisation* section.

Nothing on the release side needs a per-plugin decision, which is worth stating
once: none of the five has a `composer.json`, so the runtime inputs stay on
their defaults; none has a `nightly` or `dev` tag, so the rolling prerelease can
use the default name; all six develop on `develop`. The `npm` ecosystem
dependabot declares for maps and events is legitimate, both have a
`package.json`.

**But `maps` and `events` build their assets with npm inside `add_libs`,** and
what it produces is gitignored — `webroot/*.min.js` and friends for maps. Their
cleanup block therefore has to be **extended**, not replaced by fullcard's:
dropping the build would publish a plugin without its assets, and `git archive`
alone cannot notice. Add the npm sources (`gulpfile.js`, `package.json`,
`package-lock.json`) and `node_modules` to the list, and check the archive
carries the built bundles before dispatching anything. paypal, objectslend and
activities have no npm step.

Documentation translation, from the `.po` catalogues:

| | strings | at 100 % |
|---|---|---|
| auto | 23 | es, fr, it, pt, pt_BR, sl, ta, uk — and de at 21/23 |
| paypal | 27 | none: **25/27 everywhere**, two strings (the WPS notice) are untranslated in every language |
| maps | 25 | es, fr, it, pt, pt_BR, sl, ta, uk |
| events | 34 | es, fr, it, pt, pt_BR, sl, ta, uk |
| objectslend | 27 | es, fr, it, pt, pt_BR, sl, ta, uk |
| activities | 13 | es, fr, it, pt, pt_BR, sl, ta, uk — and de at 11/13 |

`ar`, `br`, `ca`, `nb_NO`, `oc`, `ota`, `ru`, `si`, `tr` are between empty and a
handful of strings for all six; publish a language directory only where the
catalogue is worth reading.


## 5. What a Pages workflow would buy

The sites use the legacy build, from a branch: no configuration at all, but
Jekyll 3.9, Ruby Sass 3 and only the plugins on GitHub's allow-list — which is
why this theme's SCSS is on `@import`.

Building through GitHub Actions instead would bring a modern Jekyll and dart-sass
(so `@use`, and colour functions that do not warn), any gem at all, and — the
real gain — **checks before publishing**: build, run lychee and axe, deploy only
then. Today a plugin site publishes unverified; only this theme's demo is checked
in CI. A reusable workflow in `galette-plugins/.github` would also mean a
five-line caller per site instead of a Pages setting per repository, and building
the theme from a pinned checkout rather than through `remote_theme` would make
builds reproducible and let a site be tested against an unpushed theme branch.

**What it would break**, and this is the part that decides it:
`POST /repos/{owner}/{repo}/pages/builds` only builds a site served from a
branch — the release action says so in its own warning. Switching to Actions
therefore breaks two things that currently work: the rebuild the release action
requests after a tag, and `bin/rebuild-consumers`. Both would have to become a
`workflow_dispatch` first. The workflow would also have to listen on the branch
Weblate writes to, which the legacy build handles by itself.

Nothing done today requires it. It becomes worth doing the day lychee and axe
should gate every site's publication, or a Jekyll plugin outside the allow-list
is needed — and then the two triggers get replaced before anything else.
