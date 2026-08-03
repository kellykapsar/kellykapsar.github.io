# kellykapsar.github.io

Personal academic + consulting website. Built with [Quarto](https://quarto.org),
deployed to GitHub Pages via GitHub Actions.

---

## 1. One-time setup

### 1.1 Install Quarto

Quarto is a standalone program — you do **not** need R installed unless you add
R code chunks later.

- Download from <https://quarto.org/docs/get-started/> (there's a Windows installer).
- If you use RStudio 2022.07 or newer, Quarto is already bundled, but installing
  the standalone version anyway keeps the command line working.
- Verify in a terminal:

```bash
quarto check
```

You should see `[✓] Checking Quarto installation......OK`.

### 1.2 Get the files into a repo

Your current repo is `kellykapsar/PersonalWebsite`. Because you chose
`kellykapsar.github.io` as the URL, GitHub requires the repo to be named
**exactly** `kellykapsar.github.io`.

**Option A — rename the existing repo (keeps history and stars):**

1. Go to <https://github.com/kellykapsar/PersonalWebsite/settings>
2. Under **Repository name**, change it to `kellykapsar.github.io` → **Rename**.
3. Clone or re-point your local copy:

```bash
git clone https://github.com/kellykapsar/kellykapsar.github.io.git
cd kellykapsar.github.io
```

4. The old Hugo/Wowchemy files are still in there. Move them out of the way so
   Quarto doesn't try to render them:

```bash
mkdir _old_hugo_site
git mv content themes config layouts static assets _old_hugo_site/ 2>/dev/null
# (some of those folders may not exist — that's fine)
```

Or just delete them once you're confident the new site works.

5. Copy every file from this folder into the repo root.

**Option B — start a clean repo (simpler, loses history):**

1. Create a new repo on GitHub named `kellykapsar.github.io`, empty, no README.
2. In this folder:

```bash
git init
git add .
git commit -m "New Quarto site"
git branch -M main
git remote add origin https://github.com/kellykapsar/kellykapsar.github.io.git
git push -u origin main
```

3. Archive the old `PersonalWebsite` repo (Settings → Archive) so it isn't confusing.

---

## 2. Build and preview locally

From the repo root:

```bash
# Live preview — opens a browser, auto-reloads on save. Ctrl+C to stop.
quarto preview

# One-off build into _site/
quarto render
```

`quarto preview` is what you want 95% of the time. Edit a `.qmd`, hit save, and
the browser updates.

If you use RStudio: open the folder, and there's a **Render Website** button in
the Build pane.

---

## 3. Deploy to GitHub Pages

The included GitHub Action (`.github/workflows/publish.yml`) renders the site in
the cloud on every push to `main` and publishes it to a `gh-pages` branch. You
never have to commit built HTML.

### 3.1 Create the gh-pages branch once

Quarto's publish action expects the branch to exist. Easiest way — run this
locally, one time:

```bash
quarto publish gh-pages
```

It will ask for confirmation, create the branch, push, and enable Pages. Say yes
to everything. (If it asks you to authorize, it opens a browser for GitHub login.)

### 3.2 Point GitHub Pages at it

1. Go to your repo → **Settings** → **Pages**.
2. **Source**: `Deploy from a branch`.
3. **Branch**: `gh-pages`, folder `/ (root)` → **Save**.

Your site is live at <https://kellykapsar.github.io> within a minute or two.

### 3.3 From then on

```bash
git add .
git commit -m "Update research page"
git push
```

The Action runs automatically. Watch it under the repo's **Actions** tab; a green
check means it deployed. Takes about 60–90 seconds.

### Alternative: no GitHub Actions

If you'd rather not use CI, you can render locally and serve the built files
directly:

1. In `_quarto.yml`, change `output-dir: _site` to `output-dir: docs`.
2. Remove `/_site/` from `.gitignore`.
3. Run `quarto render`, then commit and push the `docs/` folder.
4. Settings → Pages → Source: `Deploy from a branch`, branch `main`, folder `/docs`.
5. Delete `.github/workflows/publish.yml`.

This is simpler but means you must remember to render before every push.

---

## 4. Turning off the old Netlify site

Once the new site is live:

1. Log into Netlify → your `kellykapsar` site → **Site configuration** → **Danger zone**.
2. Either **Stop builds** (keeps the URL alive as a frozen archive) or **Delete site**.
3. Update your Google Scholar, LinkedIn, ORCID, and email signature to the new URL.

---

## 5. File map

```
_quarto.yml          Site config: navbar, footer, theme, metadata
styles.scss          All custom design. Palette is the first 8 lines — change
                     those five hex codes and the whole site re-themes.
index.qmd            Home page (hero, capabilities, current projects, about)
research.qmd         4 current projects + 3 past projects
publications.qmd     Selected work + full list (hand-written; see note below)
talks.qmd            Talks, outreach, public writing, teaching
cv.qmd               Web CV + link to PDF
contact.qmd          Contact details
references.bib       Full bibliography in BibTeX, offered as a download
images/              All photos. Currently PLACEHOLDERS — see IMAGES-AND-OPEN-ITEMS.md
files/               Put KapsarCV.pdf here
.github/workflows/   The deploy Action
```

### Note on publications

The publication list on `publications.qmd` is written out by hand in Markdown
rather than auto-generated from `references.bib`. That's deliberate: it means the
build can't break, and you can add a one-sentence plain-language note to any
entry — which matters more for a consulting/NGO audience than perfect CSL
formatting.

`references.bib` is kept in sync and linked as a download for people who want to
cite you. When a new paper lands, add it in both places (two minutes).

If you later decide you'd rather auto-generate, add `bibliography: references.bib`
and `nocite: |` / `  @*` to the YAML header and replace the list with `::: {#refs}
:::`. You lose per-entry commentary and section grouping.

---

## 6. Editing rule that matters

**Always leave a blank line before an opening `:::` fence.**

This is the one thing that will break the build. Pandoc absorbs an opening fence
into the paragraph above it if there's no blank line, which produces a wall of
`The following string was found in the document: :::` warnings and silently
mangles the layout.

```markdown
<!-- BROKEN — the ::: gets swallowed into the image paragraph -->
::: {.project-card}
![](images/past-ais.jpg)
::: {.pc-body}
### Title
:::
:::

<!-- CORRECT -->
::: {.project-card}
![](images/past-ais.jpg)

::: {.pc-body}
### Title
:::
:::
```

Two `:::` lines directly after each other are fine. A closing `:::` directly
after text is fine. It's only an *opening* fence following a line of text.

To check before you commit:

```bash
quarto render 2>&1 | grep -c "fenced div"
```

Zero means you're clean.

---

## 7. Images and Quarto's resource discovery

Quarto only copies files it can see in Markdown links and images. The hero photos
are referenced inside inline CSS (`background-image: url('images/...')`), which
Quarto cannot detect — so `images/` is declared explicitly under `project:
resources:` in `_quarto.yml`. If you add a new folder of assets referenced only
from CSS, add it there too or it will 404 on the deployed site.

---

## 8. Common edits

**Change the color scheme** — `styles.scss`, lines 5–12. `$accent` is the rust
highlight, `$deep` is the teal used for solid navbars and headings.

**Swap a hero photo** — drop your image in `images/` with the same filename, or
change the `background-image: url('images/...')` in the page's `.hero` block.

**Add a project** — copy an existing `.project-card` block in `research.qmd` and
edit. The grid reflows automatically.

**Add a talk** — copy a `.talk-item` block in `talks.qmd`.

**Add a nav item** — `_quarto.yml`, under `navbar: right:`.

**Reorder the nav** — same place; order in the file is order on screen.
