---
inclusion: always
---

# Documentation Standards

This is a documentation repository using MkDocs to render web pages.

## Project Structure

- `docs/` - Contains all markdown documentation files
- `mkdocs.yml` - MkDocs configuration file
- `requirements.txt` - Python dependencies including mkdocs

## Documentation Guidelines

### Writing Style
- Use clear, concise language
- Write in present tense
- Use active voice when possible
- Keep paragraphs short and focused

### Markdown Formatting
- Use proper heading hierarchy (h1 → h2 → h3)
- Include code blocks with language identifiers for syntax highlighting
- Use relative links for internal documentation references
- Add alt text for images

### File Organization
- Place all documentation in the `docs/` directory
- Use lowercase filenames with hyphens (e.g., `getting-started.md`)
- Update `mkdocs.yml` nav section when adding new pages

## MkDocs Configuration

When modifying `mkdocs.yml`:
- Maintain proper YAML indentation (2 spaces)
- Update the `nav` section to reflect documentation structure
- Test locally with `mkdocs serve` before committing

## Common Tasks

### Adding a new documentation page
1. Create markdown file in `docs/` directory
2. Add entry to `nav` section in `mkdocs.yml`
3. Use relative links from other pages if needed

### Local preview
```bash
mkdocs serve
```

### Building static site
```bash
mkdocs build
```
