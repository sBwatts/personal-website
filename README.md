# sethbwatts.com

Personal academic website for Seth Watts, PhD — Assistant Professor, School of Criminal Justice and Criminology, Texas State University.

Built with [Jekyll](https://jekyllrb.com/) and the [academicpages](https://github.com/academicpages/academicpages.github.io) theme (based on [Minimal Mistakes](https://mmistakes.github.io/minimal-mistakes/)), deployed via [GitHub Pages](https://pages.github.com/).

**Live site:** [https://www.sethbwatts.com](https://www.sethbwatts.com)

## Project Structure

```
├── _bibliography/
│   └── papers.bib             # BibTeX publication list (auto-updated by open-alex.py)
├── _pages/
│   ├── about.md               # Home / about page (includes 3 recent publications)
│   ├── publications.md        # Full publications listing (rendered from papers.bib)
│   ├── cv.md                  # CV page (links to assets/pdf/cv.pdf)
│   └── research.md            # Research projects
├── _data/
│   └── navigation.yml         # Site navigation links
├── assets/
│   ├── img/headshot.jpeg      # Profile photo
│   └── pdf/cv.pdf             # CV PDF
├── images/
│   └── headshot.jpeg          # Profile photo (used by sidebar author widget)
├── cv/
│   ├── cv.Rmd                 # CV source (R Markdown)
│   └── cv_render.R            # CV rendering script
├── data/
│   └── publications.csv       # Publications data cache
├── open-alex.py               # Fetches publications from OpenAlex API -> papers.bib + CSV
├── _config.yml                # Jekyll / academicpages site configuration
├── Gemfile                    # Ruby gem dependencies
└── .github/workflows/
    ├── deploy.yml             # Builds and deploys site on push to main
    └── update-publications.yml # Auto-syncs publications on the 1st and 15th
```

## Local Development

### Prerequisites

- [Ruby](https://www.ruby-lang.org/) 3.2+ and [Bundler](https://bundler.io/) (install via `brew install ruby`)
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

> **Note:** If your system Ruby is older than 3.2, use the Homebrew Ruby explicitly:
> ```bash
> /opt/homebrew/opt/ruby/bin/bundle exec jekyll serve
> ```

### Full build

```bash
bundle exec jekyll build
```

Output goes to `_site/` (not committed).

## Publication Pipeline

Publications are managed through two layers:

1. **Static BibTeX** (`_bibliography/papers.bib`): The source of truth for the publications page. jekyll-scholar renders this automatically on every build. The three most recent entries also appear on the home page.

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

- On every push to `main`, the workflow builds the Jekyll site and deploys it to GitHub Pages.
- No manual build step is needed — push to `main` and the site updates automatically.

### GitHub Pages settings required

In the repository Settings > Pages:
- **Source**: GitHub Actions

## Theme

The site uses [academicpages](https://github.com/academicpages/academicpages.github.io), a Jekyll theme built on Minimal Mistakes. Theme files are included directly in the repository. Key configuration:

- `_config.yml` — Site settings, author info, social links, jekyll-scholar configuration
- `_data/navigation.yml` — Top navigation links
- `_sass/layout/_page.scss` — Custom `min-height` fix to prevent footer from overlapping the sidebar on short pages
