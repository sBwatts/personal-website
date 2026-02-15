# sethbwatts.com

Personal academic website for Seth Watts, PhD — Assistant Professor at Texas State University.

Built with [Quarto](https://quarto.org/) and deployed via [GitHub Pages](https://pages.github.com/).

**Live site:** [https://www.sethbwatts.com](https://www.sethbwatts.com)

## Project Structure

```
├── index.qmd              # Homepage
├── research.qmd           # Publications listing
├── about.qmd              # About page
├── cv/
│   ├── cv.Rmd             # CV source (R Markdown)
│   └── cv_render.R        # CV rendering script
├── research/articles/     # Auto-generated article pages (from OpenAlex)
├── open-alex.py           # Syncs publications from the OpenAlex API
├── call-python.R          # R wrapper to run open-alex.py via reticulate
├── docs/                  # Rendered site output (served by GitHub Pages)
├── _quarto.yml            # Quarto project configuration
└── .github/workflows/     # GitHub Actions (automated publication updates)
```

## Local Development

### Prerequisites

- [Quarto](https://quarto.org/docs/get-started/)
- [R](https://cran.r-project.org/) with packages: `rmarkdown`, `reticulate`, `stevetemplates`
- [Python 3](https://www.python.org/) with packages: `requests`, `pyyaml`
- A LaTeX distribution (e.g., TinyTeX) for CV PDF generation

### Build the site

```bash
quarto render
```

This runs the pre-render scripts (`cv_render.R`, `call-python.R`) automatically, then renders all `.qmd` files to `docs/`.

### Preview locally

```bash
quarto preview
```

## Automated Publication Updates

A GitHub Actions workflow (`.github/workflows/update-publications.yml`) runs every two weeks to:

1. Fetch the latest publications via the [OpenAlex API](https://openalex.org/)
2. Rebuild the Quarto site
3. Commit and push any changes

The workflow can also be triggered manually from the Actions tab.

## Deployment

The site is deployed via GitHub Pages from the `docs/` folder on the `main` branch.
