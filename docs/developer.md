# Developer Information

This page contains information for developers.



## Documentation

### Prerequisites

- Python 3.x
- Git

### Setup

1. Clone the repository:
```shell
git clone https://github.com/ossrep/ossrep.github.io.git
cd ossrep.github.io
```

2. Activate the Python virtual environment:
```shell
source .venv/bin/activate
```

3. Install dependencies (if needed):
```shell
pip install -r requirements.txt
```

## Development Workflow

### Running Locally

Start the development server with live reload:
```shell
mkdocs serve
```

The site will be available at `http://127.0.0.1:8000`

Changes to markdown files will automatically reload in the browser.

### Adding New Pages

1. Create a new `.md` file in the `docs/` directory
2. Update `mkdocs.yml` to add the page to the navigation
3. Write your content using Markdown

### Building the Site

Build the static site files:
```shell
mkdocs build
```

This generates the site in the `site/` directory.

## Deployment

Deploy to GitHub Pages:
```shell
mkdocs gh-deploy
```

This command builds the site and pushes it to the `gh-pages` branch.

## Technology Stack

- **MkDocs**: Static site generator
- **Material for MkDocs**: Theme providing Material Design UI
- **Python Markdown**: Markdown processor with extensions
- **GitHub Pages**: Hosting platform

## Documentation

- [MkDocs Documentation](https://www.mkdocs.org)
- [Material for MkDocs](https://squidfunk.github.io/mkdocs-material)
