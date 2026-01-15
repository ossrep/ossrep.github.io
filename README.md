# ossrep Documentation

Documentation site for the Open Source Retail Energy Platform (ossrep).

## Local Development

### Setup

```bash
python -m venv .venv
source .venv/bin/activate
pip install mkdocs-material
```

### Serve Locally

```bash
source .venv/bin/activate
mkdocs serve --livereload
```

Visit `http://localhost:8000` to view the documentation.

### Build

```bash
mkdocs build
```

## Deployment

The documentation is automatically deployed to GitHub Pages via GitHub Actions when changes are pushed to the `main` branch.

Visit: https://ossrep.github.io
