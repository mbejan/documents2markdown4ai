# documents2markdown4ai

Convert a folder of documents into **self-contained Markdown** (images inlined as
base64 `data:` URIs) so the output can be fed directly to downstream AI agents.

The tool walks an input directory recursively, converts every supported file,
and mirrors the folder structure into an output directory.

## Features

- **Images inlined** — images are embedded as base64 data URIs, so each `.md`
  file is fully self-contained (no separate asset files to track).
- **Recursive** — mirrors the input folder structure into the output folder.
- **PDFs with images** — PDFs are extracted with [PyMuPDF](https://pymupdf.readthedocs.io/)
  (markitdown's own PDF converter is text-only), preserving both text and images.
- **Archives expanded** — `.zip` files are expanded and their contents converted
  inline (via markitdown's native archive support).
- **Broad format support** — Word, PowerPoint, Excel (`.xlsx`/`.xls`), PDF, HTML,
  CSV, JSON, XML, plain text, Markdown, and standalone images.

## Supported formats

| Category | Extensions | Notes |
|----------|------------|-------|
| Word | `.docx` | images inlined |
| PowerPoint | `.pptx` | images inlined |
| Excel | `.xlsx`, `.xls` | tables only (no images) |
| PDF | `.pdf` | text + images (PyMuPDF) |
| Web / markup | `.html`, `.htm` | |
| Data | `.csv`, `.json`, `.xml` | |
| Text | `.txt`, `.md`, `.rst` | |
| Archives | `.zip` | contents expanded inline |
| Images | `.jpg`, `.jpeg`, `.png`, `.gif`, `.bmp`, `.tiff`, `.tif`, `.webp` | embedded as data URI |

**Deliberately excluded:** audio (`.mp3`, `.wav`, …) and e-books (`.epub`).

## Requirements

- Python 3.10+ (tested on 3.14)
- Dependencies listed in `requirements.txt`

## Installation

```bash
# 1. Create and activate a virtual environment
python3 -m venv .venv
source .venv/bin/activate

# 2. Install dependencies
pip install -r requirements.txt
```

> **Note on `markitdown[all]`:** this project intentionally does **not** use the
> `[all]` extra. As of markitdown 0.1.7, `[all]` pins
> `youtube-transcript-api~=1.0.0`, a version that does not exist on PyPI (the
> project jumps from 0.6.2 to 1.2.3), which breaks `pip install markitdown[all]`.
> We install only the document-format extras we need instead.

## Usage

```bash
# Convert ./raw_documents into ./distilled_md (defaults)
python convert.py

# Specify custom directories
python convert.py --input /path/to/docs --output /path/to/markdown
```

Each input file `path/to/file.ext` becomes `path/to/file.md` in the output
directory, preserving the relative folder structure.

## Generating sample documents

To verify the pipeline end-to-end, generate a set of sample files:

```bash
python make_samples.py
```

This creates files in `./raw_documents` covering `.docx`, `.pptx`, `.xlsx`,
`.xls`, `.pdf`, `.zip`, and a standalone `.png`.

## How it works

| Input type | Converter |
|------------|-----------|
| `.docx`, `.pptx` | markitdown with `keep_data_uris=True` (images inlined) |
| `.pdf` | custom PyMuPDF extractor (text + images interleaved) |
| `.zip` | markitdown (expands entries, concatenates Markdown) |
| `.xlsx`, `.xls`, `.html`, `.csv`, `.json`, `.xml`, `.txt`, `.md`, `.rst` | markitdown default |
| standalone images | embedded directly as a base64 data URI |

## Known limitations

- **Excel images** — markitdown's Excel converters emit tables only; images
  embedded in spreadsheets are not extracted.
- **Scanned PDFs** — PyMuPDF extracts embedded images but does not OCR text.
  A scanned PDF (image-only, no text layer) will yield images without text.
  OCR (e.g. Tesseract) is a separate opt-in if needed.
- **File size** — inlining images as base64 increases `.md` file size by ~33%.
  This is intentional (self-contained output) and not a concern for this use case.