# OSSREP Documentation

Documentation site for the Open Source Retail Electric Provider platform.

**Live site:** [https://ossrep.github.io](https://ossrep.github.io)

## Local Development

### Prerequisites

- Python 3.10+
- pip

### Setup

```bash
# Install dependencies
pip install -r requirements.txt

# Serve locally with live reload
mkdocs serve
```

The site will be available at `http://localhost:8000`.

### Build

```bash
mkdocs build
```

The static site will be generated in the `site/` directory.

## Deployment

The site is automatically deployed to GitHub Pages when changes are pushed to the `main` branch.

## Adding Content

1. Add or edit Markdown files in the `docs/` directory
2. Update `mkdocs.yml` to include new pages in the navigation
3. Commit and push to `main`

## Recording Conversations

To document conversations with Claude AI:

1. Create a new file in `docs/conversations/` with the format `YYYY-MM-DD-topic.md`
2. Use the template from existing conversation files
3. Update `docs/conversations/index.md` with a link to the new conversation

## License

This documentation is part of the OSSREP project.
