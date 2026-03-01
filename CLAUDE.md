# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Quarto-based static academic website for Seth Watts (Assistant Professor, Texas State University), deployed via GitHub Pages to https://www.sethbwatts.com. Output renders to `docs/`.

## Commands

```bash
# Preview locally with live reload
quarto preview

# Full build (renders to docs/)
quarto render
```

Pre-render scripts in `_quarto.yml` run automatically before every render:
1. `Rscript cv/cv_render.R` — compiles `cv/cv.Rmd` to `cv/cv.pdf` via R/TinyTeX
2. `Rscript call-python.R` — calls `open-alex.py` via reticulate to sync publications from OpenAlex API

## Architecture

### Publication Pipeline

`open-alex.py` fetches publications from the OpenAlex API using ORCID `0000-0002-5108-9055`, writes data to `data/publications.csv` and `data/publications.rds`, and generates individual `.qmd` files in `research/articles/`. These are picked up automatically by Quarto. `research.qmd` lists all articles; `index.qmd` shows the latest 3.

`call-python.R` is the R wrapper that invokes `open-alex.py` via reticulate. The GitHub Actions workflow (`.github/workflows/update-publications.yml`) runs this pipeline automatically on the 1st and 15th of each month, then commits and pushes any changes.

### CV Pipeline

`cv/cv.Rmd` is the CV source (R Markdown using `stevetemplates`). `cv/cv_render.R` renders it to `cv/cv.pdf`, which is included as a resource in the final site. The `cv/` directory is excluded from Quarto's `.qmd` rendering (`!cv/**` in `_quarto.yml`).

### Key Files

| File | Purpose |
|------|---------|
| `_quarto.yml` | Project config: navbar, pre-render hooks, output dir, resources |
| `styles.css` | Custom CSS with variables for colors/typography (Inter, JetBrains Mono) |
| `open-alex.py` | Fetches publications from OpenAlex; generates `research/articles/*.qmd` |
| `cv/cv.Rmd` | CV source document |
| `research/articles/` | Auto-generated — do not edit manually |
| `docs/` | Rendered site — do not edit manually |

### Dependencies

- **Quarto** for rendering
- **Python**: `requests`, `pyyaml` (see `requirements.txt`)
- **R**: `rmarkdown`, `reticulate`, `stevetemplates`
- **TinyTeX/LaTeX** for CV PDF generation

## Git Workflow

All changes to this repository must be committed and pushed to GitHub with clear, descriptive commit messages.

### Commit Message Guidelines

- Use the imperative mood: "Add feature" not "Added feature"
- Keep the first line under 72 characters
- Describe *what* changed and *why*, not *how*
- Examples:
  - `Fix 404 on de-escalation article by adding missing rendered HTML`
  - `Update publications: add comparative de-escalation review (2026-02-16)`
  - `Update CV to reflect new publications`

### Push Workflow

```bash
# Stage specific files (never use git add -A blindly)
git add <file1> <file2>

# Commit with a clear message
git commit -m "Short descriptive message explaining the change"

# Push to the current branch
git push -u origin <branch-name>
```

### Important Rules

- **Always push** after committing — local-only commits are not deployed
- **Never force-push** to `main` without explicit confirmation
- **Never skip hooks** (`--no-verify`) unless explicitly requested
- The `docs/` directory is the deployed site — changes there go live when pushed to the branch tracked by GitHub Pages
