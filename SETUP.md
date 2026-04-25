# Personal Website — Setup & Editing Guide

Everything you need to run, customize, and grow this site.

---

## Running the site locally

**First time only — install dependencies:**
```bash
cd /path/to/this/repo
bundle install
```

**Start the dev server:**
```bash
bundle exec jekyll serve
```
Open [http://localhost:4000](http://localhost:4000). The site rebuilds automatically when you save a file.

**Useful flags:**
```bash
bundle exec jekyll serve --livereload   # auto-refresh browser on file save
bundle exec jekyll serve --drafts       # also show draft posts from _drafts/
```

---

## Things you still need to fill in

| What | File | What to change |
|------|------|----------------|
| Your photo | `assets/img/prof_pic.png` | Replace with your own photo (same filename) |
| Your email | `_config.yml` + `_pages/about.md` | Replace `your@email.com` |
| LinkedIn handle | `_config.yml` → `linkedin_username` | e.g. `bilal-naseem` |
| Medium handle | `_config.yml` → `medium_username` | e.g. `bilalnaseem` |
| Your location | `_pages/about.md` → profile address | Replace `[Your City, State]` |
| Paper 1 details | `_bibliography/papers.bib` | Replace first `@inproceedings` block |
| Paper 2 details | `_bibliography/papers.bib` | Replace second `@inproceedings` block |
| Real experience | `_data/cv.yml` → Experience section | Replace the placeholder entries |
| Your CV PDF | `assets/pdf/Bilal_Naseem_CV.pdf` | Upload your PDF here |
| Project GitHub links | `_projects/*.md` → `github:` field | Replace with real repo URLs (or remove the field) |

---

## Editing the home page bio

File: `_pages/about.md`

The content below the `---` frontmatter is plain Markdown. Edit freely.

To change what sections appear on the front page, toggle the frontmatter flags:
```yaml
news: true             # show the Highlights section
featured_projects: true  # show Featured Projects
selected_papers: false   # show Selected Publications (set true if you want papers shown)
social: true           # show social icons at the bottom
```

To change your profile sidebar (photo, location, availability badge):
```yaml
profile:
  align: right           # photo on right; use "left" to flip
  image: prof_pic.png    # filename inside assets/img/
  image_circular: false  # true = round crop
  address: >
    <p>📍 Your City, State</p>
    <p>🟢 Open to Opportunities</p>
```

---

## Adding a blog article

1. Create a file in `_posts/` named exactly `YYYY-MM-DD-your-title.md`
2. Add this frontmatter at the top:

```yaml
---
layout: post
title: "Your Article Title"
date: 2026-05-01
description: One-sentence summary shown in the article list.
tags: [spark, data-engineering, python]
categories: data-engineering
---

Your article in Markdown starts here...
```

The article will appear at `/blog/` and in the nav under **articles**.

**To write a draft** (not published yet), put the file in `_drafts/` without a date prefix:
```
_drafts/my-draft-article.md
```
Run `bundle exec jekyll serve --drafts` to preview it.

---

## Adding a project

1. Create `_projects/7_project.md` (number controls sort order — lower = appears first)
2. Use this template:

```yaml
---
layout: page
title: Your Project Name
description: One-line description shown on the project card.
img: assets/img/your-image.jpg   # thumbnail; remove line if no image
importance: 7                    # lower number = sorted first within category
category: data-engineering       # or: machine-learning
featured: true                   # true = also shows on the home page
github: https://github.com/...   # optional; shows GitHub icon on card
---

## Overview

Describe the project here in Markdown. You can use headers, bullet points, code blocks, etc.

## Tech Stack

`Python` `Spark` `AWS`
```

**Categories** must match what's in `_pages/projects.md` → `display_categories`. Current values: `data-engineering`, `machine-learning`.

**Featured projects** (the 3 shown on the home page) are controlled by `featured: true` in the frontmatter. You can feature as many as you want; they'll all show. Currently projects 1, 2, and 3 are featured.

---

## Adding a paper / publication

Open `_bibliography/papers.bib` and add a BibTeX entry:

```bibtex
@inproceedings{naseem2026paper,
  abbr={NeurIPS},           # short label shown as a badge
  bibtex_show={true},       # allow BibTeX copy button
  title={Your Paper Title},
  author={Naseem, Bilal and Coauthor, Name},
  booktitle={Conference Full Name},
  year={2026},
  arxiv={2xxx.xxxxx},       # optional: links to arXiv
  pdf={paper.pdf},          # optional: put file in assets/pdf/
  code={https://github.com/...},  # optional
  selected={true}           # true = shows on home page (requires selected_papers: true in about.md)
}
```

Then add the year to the `years:` list in `_pages/publications.md` if it's a new year:
```yaml
years: [2026, 2025, 2024, 2023]
```

---

## Editing the CV

File: `_data/cv.yml`

The CV is structured YAML. There are four section types:

### `map` — key/value pairs (used for General Information)
```yaml
- title: General Information
  type: map
  contents:
    - name: Full Name
      value: Bilal Naseem
```

### `time_table` — timeline entries (used for Education, Experience)
```yaml
- title: Experience
  type: time_table
  contents:
    - title: Senior Data Engineer
      institution: Acme Corp
      year: 2023 – Present
      description:
        - Built X, resulting in Y% improvement
        - Led Z initiative across N teams
```

### `nested_list` — grouped bullet lists (used for Skills)
```yaml
- title: Skills
  type: nested_list
  contents:
    - title: Data Engineering
      items:
        - Apache Spark, Kafka, Airflow, dbt
        - Snowflake, BigQuery, Delta Lake
```

### `list` — flat bullet list
```yaml
- title: Certifications
  type: list
  contents:
    - AWS Certified Data Engineer – Associate
    - Google Professional Data Engineer
```

**To add a CV PDF download button:** place your PDF at `assets/pdf/Bilal_Naseem_CV.pdf` — the button in the CV page header will activate automatically.

---

## Adding a news/highlight item

Files: `_news/announcement_N.md`

These are the items in the **Highlights** section on the home page. The most recent 5 are shown (controlled by `news_limit: 5` in `_config.yml`).

Template:
```yaml
---
layout: post
date: 2026-05-01 09:00:00-0400
inline: true
---

Your announcement here. You can use **Markdown** and [links](https://example.com).
```

The items are sorted by date — newest at the top. Add new announcements by creating `announcement_11.md`, `announcement_12.md`, etc., or by overwriting old placeholder entries.

---

## Changing the color theme

File: `_sass/_themes.scss`

The primary accent color (purple by default) is defined near the top:
```scss
--global-theme-color: #B509AC;    // light mode accent
```
And in the dark section:
```scss
--global-theme-color: #2698BA;    // dark mode accent
```
Change the hex values to any color you want.

---

## Social links (navbar + about page footer)

All social links are controlled by setting the relevant username in `_config.yml`. Set a field to blank to hide that icon:

```yaml
github_username: BilalNaseem1
linkedin_username: bilal-naseem
medium_username: bilalnaseem
twitter_username:             # blank = hidden
kaggle_id: bilalnaseem        # shows Kaggle icon if set
```

The icons appear in two places:
- **Top-left navbar** (on the home page only) — controlled by `enable_navbar_social: true` in `_config.yml`
- **Bottom of home page bio** — controlled by `social: true` in `_pages/about.md`

---

## Navigation bar items

The nav is controlled by the `nav: true` frontmatter in each page file. Current nav pages and their order:

| Page | File | `nav_order` |
|------|------|-------------|
| about | `_pages/about.md` | (always first, auto) |
| articles (blog) | auto from `_config.yml` → `blog_nav_title` | (always second) |
| portfolio | `_pages/projects.md` | 2 |
| cv | `_pages/cv.md` | 3 |
| research | `_pages/publications.md` | 4 |

To hide a page from the nav: set `nav: false` in its frontmatter.
To reorder: change the `nav_order` number.

---

## Deploying to GitHub Pages

This repo uses a **GitHub Actions** workflow (`.github/workflows/deploy.yml`) that automatically builds the site and pushes it to GitHub Pages on every push to `master`. You do not need to build manually.

### Step 1 — Create the GitHub repo

1. Go to [github.com/new](https://github.com/new)
2. Name it exactly **`bilalnaseem1.github.io`** (must match your username)
3. Set it to **Public**
4. Do **not** initialise with a README (you're pushing existing code)

### Step 2 — Update the site URL

In `_config.yml`, make sure this line is correct:
```yaml
url: https://bilalnaseem1.github.io
```

### Step 3 — Push the code

```bash
cd /path/to/this/repo

# Point your local repo at the new GitHub remote
git remote set-url origin https://github.com/BilalNaseem1/bilalnaseem1.github.io.git
# (if "set-url" errors, use "add" instead: git remote add origin ...)

git add .
git commit -m "initial personal website"
git push -u origin master
```

### Step 4 — Enable GitHub Pages

1. Go to your repo on GitHub → **Settings** → **Pages**
2. Under **Source**, select **Deploy from a branch**
3. Set the branch to **`gh-pages`** and folder to **`/ (root)`**
4. Click **Save**

> The first time you push, the Actions workflow will create the `gh-pages` branch automatically. If Pages says the branch doesn't exist yet, wait ~2 minutes for the workflow to finish, then refresh the Settings page.

### Step 5 — Watch it build

- Go to the **Actions** tab in your repo
- You'll see a `deploy` workflow running — it takes about 2–3 minutes
- Once it shows a green tick, your site is live at `https://bilalnaseem1.github.io`

### Every future update

Just push to `master` — the workflow rebuilds and deploys automatically:
```bash
git add .
git commit -m "your update message"
git push
```

### Troubleshooting

| Problem | Fix |
|---------|-----|
| Build fails in Actions | Click the failed run → expand the red step → read the error log |
| Site shows old content | Hard-refresh the browser (`Ctrl+Shift+R`) or wait a minute for CDN cache |
| `gh-pages` branch not created | Manually trigger the workflow: Actions → deploy → Run workflow |
| `bundle install` fails | Delete `Gemfile.lock` and retry |
| Pages shows 404 | Check Settings → Pages — branch must be `gh-pages`, not `master` |

### Custom domain (optional)

1. Buy a domain (e.g. `bilalnaseem.dev`)
2. In your domain registrar, add a CNAME record: `www` → `bilalnaseem1.github.io`
3. In repo Settings → Pages → Custom domain, enter your domain
4. Check **Enforce HTTPS**
5. Update `_config.yml`:
```yaml
url: https://www.bilalnaseem.dev
```
