# sethbwatts.com

Personal academic website for Seth Watts, PhD — Assistant Professor, School of Criminal Justice and Criminology, Texas State University.

Built with [Jekyll](https://jekyllrb.com/) and the [al-folio](https://github.com/alshedivat/al-folio) theme, deployed via [GitHub Pages](https://pages.github.com/).

**Live site:** [https://www.sethbwatts.com](https://www.sethbwatts.com)

## Project Structure

```
├── _bibliography/
│   └── papers.bib             # BibTeX publication list (auto-updated by open-alex.py)
├── _pages/
│   ├── about.md               # Home / about page
│   ├── publications.md        # Publications listing (rendered from papers.bib)
│   ├── cv.md                  # CV page (links to assets/pdf/cv.pdf)
│   └── projects.md            # Research projects listing
├── _projects/
│   ├── truleo.md              # Truleo AI BWC evaluation project
│   └── orp.md                 # Tempe Opioid Recovery Project
├── assets/
│   ├── img/prof_pic.jpg       # Profile photo
│   └── pdf/cv.pdf             # CV PDF
├── cv/
│   ├── cv.Rmd                 # CV source (R Markdown)
│   └── cv_render.R            # CV rendering script
├── data/
│   └── publications.csv       # Publications data cache
├── open-alex.py               # Fetches publications from OpenAlex API -> papers.bib + CSV
├── call-python.R              # Thin R wrapper to run open-alex.py
├── _config.yml                # Jekyll / al-folio site configuration
├── Gemfile                    # Ruby gem dependencies
└── .github/workflows/
    ├── deploy.yml             # Builds and deploys site on push to main
    └── update-publications.yml # Auto-syncs publications on the 1st and 15th
```

## Local Development

### Prerequisites

- [Ruby](https://www.ruby-lang.org/) 3.2+ and [Bundler](https://bundler.io/)
- [Python 3](https://www.python.org/) with packages: `requests`, `pyyaml`

### Install dependencies

```bash
bundle install
```

### Preview locally

```bash
bundle exec jekyll serve
```

The site will be available at `http://localhost:4000`.

### Full build

```bash
bundle exec jekyll build
```

Output goes to `_site/` (not committed).

## Publication Pipeline

Publications are managed through two layers:

1. **Static BibTeX** (`_bibliography/papers.bib`): The source of truth for the publications page. jekyll-scholar renders this automatically on every build.

2. **Auto-sync** (`open-alex.py`): Fetches the latest publications from the [OpenAlex API](https://openalex.org/) using ORCID `0000-0002-5108-9055`, then writes/overwrites `_bibliography/papers.bib` and `data/publications.csv`.

To manually sync publications locally:

```bash
python open-alex.py
```

The GitHub Actions workflow (`.github/workflows/update-publications.yml`) runs this automatically on the 1st and 15th of each month, then commits and pushes any changes.

## CV Updates

The CV is a PDF generated from `cv/cv.Rmd` using R and TinyTeX. To update:

1. Edit `cv/cv.Rmd`
2. Run `Rscript cv/cv_render.R` to produce a new `cv/cv.pdf`
3. Copy the updated PDF to `assets/pdf/cv.pdf`
4. Commit and push both files

## Deployment

The site is deployed via GitHub Actions using the `deploy.yml` workflow:

- On every push to `main`, the workflow builds the Jekyll site and deploys it to GitHub Pages via the `gh-pages` environment.
- No manual build step is needed — push to `main` and the site updates automatically.

### GitHub Pages settings required

In the repository Settings > Pages:
- **Source**: Deploy from a branch OR GitHub Actions (if using the Actions workflow)
- Set to serve from the `gh-pages` environment created by the workflow

## Theme customization

The site uses [al-folio](https://github.com/alshedivat/al-folio) via `remote_theme`. Custom overrides:

- `_sass/_custom.scss` — Font family overrides (Inter, JetBrains Mono)
- `_includes/head_custom.html` — Google Fonts loading
- `_config.yml` — All site settings, social links, scholar configuration
