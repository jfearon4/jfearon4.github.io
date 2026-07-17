# CLAUDE.md

Context for Claude Code sessions working on this repo. This is James Fearon's academic
website — a fork of the [academicpages](https://github.com/academicpages/academicpages.github.io)
Jekyll template, hosted at https://jfearon4.github.io via GitHub Pages.

## Local development environment

- System Ruby (macOS default) is too old for this site's gems. Modern Homebrew Ruby
  (4.x) is *too new* — several gems this template needs (e.g. `rouge`) don't support
  it yet, and it breaks stdlib assumptions `jekyll` 3.9 relies on (e.g. `csv`).
  **Use Ruby 3.2** via `brew install ruby@3.2`. PATH is already set up in `~/.zprofile`:
  `export PATH="/opt/homebrew/opt/ruby@3.2/bin:$PATH"`.
- To preview locally: `cd` into this repo, run `bundle exec jekyll serve`, open
  `http://localhost:4000`. Auto-rebuilds on file save — **except** `_config.yml`
  changes, which require stopping (Ctrl+C) and restarting the server.
- GitHub Desktop is used for git commit/push. It is not a file manager — it can't
  create new files or folders itself; that has to happen via Finder, a text editor,
  or Terminal first, then GitHub Desktop picks up the changes to commit.
- Git doesn't track empty directories — a new folder needs at least one file in it
  before it'll show up as a change in GitHub Desktop.

## Deployment

- This repo builds via **GitHub Actions**, not the legacy built-in GitHub Pages
  Jekyll builder (that legacy builder became unreliable/deprecated and was the
  source of a silent build-failure incident — see `.github/workflows/jekyll.yml`).
- Repo setting **Settings → Pages → Build and deployment → Source** must stay set to
  "GitHub Actions".
- If a live-site change doesn't appear after pushing, check the repo's **Actions**
  tab for a failed run before assuming it's a caching/propagation delay.

## Common build-breaking mistakes (all have bitten this project before)

- `date:` as a bare integer (e.g. `date: 2026`) breaks Jekyll — YAML parses it as a
  number, not a date. Always use a quoted or `YYYY-MM-DD` value.
- **Duplicate `permalink:` values** across any two files (even across different
  collections) cause a fatal build error. Every `permalink:` in the whole site must
  be unique.
- `paperurl: ''` (empty string) is still "truthy" in Liquid, so a `[PDF]`/`[pdf]`
  link will still render with a dead link. To hide the link, **delete the
  `paperurl:` line entirely** rather than blanking it.

## Publications (`_publications/`)

One Markdown file per entry, named `YYYY-MM-DD-slug.md`. Site auto-sorts by this
date (newest first). Front matter fields:

- `title`, `collection: publications`
- `category`: must be one of the keys under `publication_category` in `_config.yml`
  — currently `books`, `manuscripts` ("Journal Articles"), `editedvolumes`
  ("Papers in Edited Volumes"), `comments` ("Published Comments"). Anything else
  silently doesn't appear anywhere on the Publications page.
- `permalink`: unique across the whole site
- `date`: real date, see above
- `venue`: informational only, not directly rendered by the current templates
- `paperurl` (optional): `/files/<name>.pdf` — omit entirely if no PDF yet
- `citation`: this is the actual visible text on the list. Supports Markdown
  (`[title](url)` for the journal webpage link) and raw HTML (e.g. `<i>` for
  italics) — it's piped through `markdownify` in the include.

Rendering: `_pages/publications.html` groups entries by category under a heading
per category, with a jump-nav at the top that **only shows links to categories
that actually have entries** (avoids dead anchors). Each entry renders via
`_includes/archive-single-publication.html`: title is a clickable link (to the
journal's own page, filled in manually), solo-authored papers show no author name,
co-authored ones show `(co-authored with X)` after the title, and there's a `[PDF]`
link if `paperurl` is set.

PDFs live in the top-level `files/` folder.

## Unpublished (`_unpublished/`)

Same one-file-per-entry pattern as Publications, but deliberately different in a
few ways:

- **Ordering is manual**, via an `order:` front-matter field (integer, fractional
  values like `5.5` are fine for inserting between existing entries) — `date`
  isn't used for sorting here since many entries are undated working papers.
  `_pages/unpublished.html` sorts by `order` ascending.
- Titles are **plain text, not hyperlinked** (this was an explicit style choice,
  different from Publications).
- Each entry shows `[pdf]` (if `paperurl` set) and `[abstract]` (only appears if
  the file has body content below the front matter — the abstract text goes there,
  and the link points to the entry's own auto-generated subpage).
- Renders via `_includes/archive-single-unpublished.html`.

## Other pages

- `_data/navigation.yml` controls the top header links — list of `title`/`url`
  pairs, in display order.
- `_pages/cv.md` still has the original template's placeholder content (fake
  "GitHub University" degrees, etc.) — not yet replaced with real content.
- `images/` is the real source folder for images (referenced e.g. by
  `author.avatar` in `_config.yml`). `_site/` (including `_site/images/`) is
  Jekyll's generated build output, is git-ignored, and should never be edited
  directly — it gets wiped and regenerated on every build.
