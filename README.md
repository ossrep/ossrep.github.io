# ossrep.github.io

OSSREP organization documentation site built with [Material for MkDocs](https://squidfunk.github.io/mkdocs-material).

## Prerequisites

- Python 3.x

## Local Development

Create and activate a virtual environment, then install dependencies:

```shell
python3 -m venv .venv
source .venv/bin/activate
pip install mkdocs-material
```

Start the development server:

```shell
source .venv/bin/activate
mkdocs serve --livereload
```

The site will be available at [http://localhost:8000](http://localhost:8000).

## Deployment

The site is automatically deployed to GitHub Pages via GitHub Actions on push to `main`.
