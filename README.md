# Account sale pipeline

End-to-end tooling that reads buyer account-sale PDFs (or pre-extracted `.txt`), detects the document type, parses structured revenue and cost lines, and writes standardized CSV/XLSX outputs plus optional upload-format files.

## What it does

1. **Extract text** from PDFs in an input folder (PyMuPDF by default; selective Google Document AI OCR for certain filenames—see pipeline code).
2. **Detect buyer / layout** from text and route to the matching parser (Global Fruit Point / Macalea, EDEKA, Origin Fruit, Aartsen, Grapes Direct, Everest, EXSA, etc.).
3. **Emit outputs** under your chosen output directory: raw and standardized CSVs, `revenue_output` / `cost_output`, `revenue_upload` / `cost_upload`, and `debug_records.json` by default.

Mapping rules live in `account_sale_pipeline/config/` (default: `mapping_buyer_docs.xlsx`).

## Requirements

- Python 3.10+ recommended  
- Dependencies: `account_sale_pipeline/requirements.txt` (pandas, openpyxl, PyMuPDF, optional Document AI / OpenAI / DB drivers depending on features you use).

## Setup

From the **repository root** (the folder that contains `account_sale_pipeline/`):

```bash
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r account_sale_pipeline/requirements.txt
```

Optional: set environment variables or use `account_sale_pipeline/.credentials/.env` for Document AI, OpenAI, or DB-backed features—never commit real secrets.

## Run the CLI

Default input folder is `account_sale_pipeline/input_documents`; default output is `account_sale_pipeline/output` when using that default input path.

```bash
python -m account_sale_pipeline.cli
```

Useful options:

| Option | Purpose |
|--------|---------|
| `--txt-dir PATH` | Folder with PDFs and/or `.txt` files |
| `--out-dir PATH` | Override output directory |
| `--mapping PATH` | Custom mapping CSV/XLSX |
| `--ocr-used` | Flag outputs when text came from OCR |
| `--no-write-debug` | Skip `debug_records.json` |
| `--no-standardize` | Skip `output_by_document_standardized.csv` |

Example with explicit folders:

```bash
python -m account_sale_pipeline.cli \
  --txt-dir ./account_sale_pipeline/input_documents \
  --out-dir ./account_sale_pipeline/output
```

## Outputs (typical)

Written to the output directory (unless `.gitignore` excludes them locally):

- `output_by_document.csv` — raw combined rows  
- `revenue_output.csv` / `.xlsx`, `cost_output.csv` / `.xlsx`  
- `revenue_upload` / `cost_upload` — formatted for downstream upload  
- `output_by_document_standardized.csv` — standardized layout  
- `debug_records.json` — per-document parse/debug payload  

## Optional tooling

- **`gold_agent.py`** — LLM-assisted comparison to gold spreadsheets for tuning parsers (needs API keys as documented in that script).  
- Run from repo root: `python -m account_sale_pipeline.gold_agent` (see file docstring).

## Project layout

```
account_sale_pipeline/
  cli.py              # Entrypoint
  extract.py          # Orchestration: PDF → text → routing
  detect.py           # Document type detection
  export.py           # CSV/XLSX writers
  buyers_*.py         # Buyer-specific parsers
  config/             # Mapping workbook(s)
  input_documents/    # Default PDF input (local)
  output/             # Default output (ignored by git if listed in .gitignore)
```

## License

Add a license file if this repository is shared publicly or across teams.
