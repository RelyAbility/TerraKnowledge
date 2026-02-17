# AGENTS — Exporting Terra Markdown to PDF

This repo uses `pandoc` for exports.

## Prerequisites (macOS)

Pandoc is required:
- Install: `brew install pandoc`
- Check: `pandoc --version`

PDF output requires a LaTeX engine. The simplest path on macOS is MacTeX:
- Install: `brew install --cask mactex-no-gui`
- Check (one of these should exist): `xelatex --version` or `pdflatex --version`

## Export a single file to PDF

From the repo root:

- Basic export:
  - `pandoc "md/Terra/00_Strategy_&_Vision/HHI_Executive_Summary_v1_0.md" -o "exports/HHI_Executive_Summary_v1_0.pdf"`

- With table of contents:
  - `pandoc "md/Terra/00_Strategy_&_Vision/HHI_Executive_Summary_v1_0.md" --toc -o "exports/HHI_Executive_Summary_v1_0.pdf"`

- Use a robust PDF engine (recommended):
  - `pandoc "md/Terra/00_Strategy_&_Vision/HHI_Executive_Summary_v1_0.md" --pdf-engine=xelatex -o "exports/HHI_Executive_Summary_v1_0.pdf"`

Tip: Create the output folder once:
- `mkdir -p exports`

## Export an entire folder (batch)

Export every `.md` under a folder to matching PDFs in `exports/`, preserving subfolders:

```bash
mkdir -p exports

base="md/Terra/00_Strategy_&_Vision"
find "$base" -type f -name "*.md" -print0 | while IFS= read -r -d '' f; do
  rel="${f#md/Terra/}"
  out="exports/${rel%.md}.pdf"
  mkdir -p "$(dirname "$out")"
  pandoc "$f" --pdf-engine=xelatex -o "$out"
done
```

## Export everything under md/Terra

```bash
mkdir -p exports

find "md/Terra" -type f -name "*.md" -print0 | while IFS= read -r -d '' f; do
  rel="${f#md/Terra/}"
  out="exports/${rel%.md}.pdf"
  mkdir -p "$(dirname "$out")"
  pandoc "$f" --pdf-engine=xelatex -o "$out"
done
```

## Common options

- Add a table of contents: `--toc`
- Set PDF engine explicitly: `--pdf-engine=xelatex`
- Set metadata title: `--metadata title="Terra"`

Example:
- `pandoc "md/Terra/04_Technical_Architecture/HHI_Technical_Architecture_v1_0.md" --toc --metadata title="HHI Technical Architecture" --pdf-engine=xelatex -o "exports/HHI_Technical_Architecture_v1_0.pdf"`
