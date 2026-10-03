# zume-editor-help

User-facing help and documentation for **Zume Editor**, published with
[MkDocs Material](https://squidfunk.github.io/mkdocs-material/) to GitHub Pages:

- Live site: <https://zume-editor-dev.github.io/zume-editor-help/>
- Issues (bug reports / questions): <https://github.com/zume-editor-dev/zume-editor-help/issues>

## Languages

Bilingual (English + Japanese) via
[`mkdocs-static-i18n`](https://ultrabug.github.io/mkdocs-static-i18n/) using the
**suffix** structure. The default language is English; Japanese lives beside it:

```
docs/installation.md        # English (default)
docs/installation.ja.md     # 日本語
```

Add a language later by adding a `locale` block in `mkdocs.yml` and `*.<locale>.md`
files. The language switcher appears automatically in the header.

## Edit / preview locally

Requires Python 3.9+.

```bash
python -m venv .venv
. .venv/Scripts/activate      # Windows ;  . .venv/bin/activate on macOS/Linux
pip install -r requirements.txt
mkdocs serve                  # http://127.0.0.1:8000/
```

Edit the Markdown in `docs/`. Every English page `foo.md` should have a Japanese
`foo.ja.md`. Keep the two in sync when you change one.

## Deploy

Pushing to `main` runs `.github/workflows/deploy.yml`, which builds the site and
pushes it to the `gh-pages` branch with `mkdocs gh-deploy`. One-time repo setup:

1. **Settings → Pages → Build and deployment → Source: Deploy from a branch**,
   branch **`gh-pages`** / `/ (root)`.
2. Push to `main` (or run the *deploy* workflow manually) and wait for the action
   to finish.

To deploy by hand instead: `mkdocs gh-deploy --force`.

## Structure

The navigation is defined in `mkdocs.yml` (`nav:`). Section group labels are
translated for Japanese via the `nav_translations` map; page titles come from
each page's `# H1`, so they localise automatically.
