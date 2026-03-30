# C++ Alliance WG21 Papers

This repository contains papers written for the ISO C++ Standards Committee (WG21) by C++ Alliance.

## Repository Structure

- `source/` - Current paper drafts (markdown)
- `archive/` - Previous revisions
- `tools/` - Build scripts and styling

## Building Papers

From `source/` or `archive/`:

```bash
make <paper>.html   # Generate HTML
make <paper>.pdf    # Generate PDF (requires LaTeX)
```

Output files are created in the repository root.

## Requirements

- pandoc
- mermaid-filter (`npm install -g mermaid-filter`)
- xelatex (for PDF output)

Run `make check` to verify dependencies.

