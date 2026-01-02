# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a static documentation site built with MkDocs and Material for MkDocs theme, hosted on GitHub Pages at https://ossrep.github.io. The site uses a Python virtual environment for dependency management.

## Environment Setup

The project uses a Python virtual environment located at `.venv/`. Before running any commands:

```shell
source .venv/bin/activate
```

To deactivate when done:
```shell
deactivate
```

## Common Commands

**Local Development Server:**
```shell
mkdocs serve
```
This starts a live-reloading server for local preview.

**Build Documentation:**
```shell
mkdocs build
```
Generates static site files for deployment.

**Deploy to GitHub Pages:**
```shell
mkdocs gh-deploy
```
Builds and pushes the site to the gh-pages branch (via ghp-import).

## Architecture

- **mkdocs.yml**: Main configuration file defining site name, URL, and theme
- **docs/**: Contains all markdown documentation files
  - **index.md**: Homepage of the documentation site
- **requirements.txt**: Python dependencies (MkDocs, Material theme, extensions)
- **.venv/**: Python virtual environment (gitignored)

The site uses Material for MkDocs theme which provides enhanced UI/UX features. All content is written in Markdown and processed by MkDocs into a static site.

## Key Dependencies

- **mkdocs**: Core static site generator
- **mkdocs-material**: Material Design theme
- **pymdown-extensions**: Markdown extensions for enhanced formatting
- **ghp-import**: GitHub Pages deployment tool
