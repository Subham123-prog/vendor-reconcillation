# vendor-reconcillation

Vendor Reconcillation (Vendor Reconciliation)
A lightweight Python tool to reconcile vendor invoices, purchase orders, and payments across systems to help detect mismatches, missing invoices, or duplicate payments.

Note: The repository name contains a likely typo ("reconcillation" vs "reconciliation"). Consider renaming the repo to improve discoverability.

Table of contents
- About
- Features
- Requirements
- Installation
- Quick start
- Configuration
- Input formats
- How reconciliation works (high level)
- Output
- Examples
- Tests
- Contributing
- Troubleshooting
- License
- Contact

About
This project provides utilities and scripts to compare records from different vendor-related sources (e.g., accounting exports, ERP reports, CSV/Excel dumps) and produce a reconciliation report. It is intended as a starting point for automating vendor reconciliations in Python and can be extended to integrate with real systems.

Features
- Normalize and parse common vendor file formats (CSV, Excel).
- Matching heuristics for invoice numbers, amounts, and dates.
- Detects duplicates, missing records, and amount mismatches.
- Generates human-readable reconciliation reports (CSV/Excel/JSON).
- Configurable matching rules and tolerance thresholds.

Requirements
- Python 3.8+
- Recommended packages (example):
  - pandas
  - openpyxl
  - pyyaml

Installation
1. Clone the repository:
   git clone https://github.com/Subham123-prog/vendor-reconcillation.git
2. Create and activate a virtual environment:
   python -m venv .venv
   source .venv/bin/activate  # macOS/Linux
   .venv\Scripts\activate     # Windows
3. Install dependencies:
   pip install -r requirements.txt
   (If requirements.txt is not present, install pandas and openpyxl as needed: pip install pandas openpyxl pyyaml)

Quick start
Command-line (example):
python -m vendor_reconcillation.cli --left data/accounting_export.csv --right data/vendor_statements.xlsx --config config/reconcile.yml --out reports/reconciliation_report.xlsx

Python API (example):
from vendor_reconcillation import reconciler, io

left = io.load_file("data/accounting_export.csv")
right = io.load_file("data/vendor_statements.xlsx")

cfg = reconciler.load_config("config/reconcile.yml")
result = reconciler.reconcile(left, right, cfg)
result.to_excel("reports/reconciliation_report.xlsx")

Configuration
A YAML or JSON configuration lets you define:
- which columns represent invoice id, vendor id, date, and amount
- tolerances for amount matching (absolute or percentage)
- date fuzziness (days)
- priority columns to use when matching
- output formats

Example config (config/reconcile.yml):
match:
  invoice_column: invoice_no
  vendor_column: vendor_id
  date_column: date
  amount_column: amount
  amount_tolerance:
    type: percent
    value: 0.5   # 0.5% tolerance
  date_tolerance_days: 3

Input formats
Supported (out of the box):
- CSV (.csv)
- Excel (.xls, .xlsx)

Each input should contain at least:
- an invoice number or reference
- a vendor identifier (name or id)
- an amount
- a date (invoice or transaction date)

How reconciliation works (high level)
1. Load and normalize both datasets (trim whitespace, unify date formats, convert numeric types).
2. Optionally apply vendor name normalization or mapping.
3. Attempt exact matches on invoice number and vendor.
4. For unmatched records, apply fuzzy matching using amount/date heuristics:
   - match by vendor + amount within tolerance + date within tolerance
   - flag duplicates (same invoice number + different amounts)
5. Produce a report with categories:
   - Matched
   - Left-only (present in left dataset only)
   - Right-only (present in right dataset only)
   - Amount mismatch
   - Duplicates

Output
Reconciliation outputs include:
- A detailed report listing matched and unmatched records (CSV/Excel/JSON)
- Summary statistics (counts of matches, mismatches, totals)
- Optional logs for manual review

Examples
- Reconcile Accounts Payable export with vendor statements:
  python -m vendor_reconcillation.cli --left ap_export.csv --right vendor_statements.xlsx --config config/reconcile.yml
- Run a quick check returning JSON:
  python -m vendor_reconcillation.cli --left a.csv --right b.csv --out report.json

Tests
If tests exist, run them with:
pytest

If no tests are present yet, add unit tests under tests/ using pytest and test critical functions (parsing, matching, tolerance logic).

Contributing
Contributions are welcome:
- Open an issue for bugs or feature requests.
- Follow repository coding style and include tests for new behavior.
- Submit pull requests against the main branch.

Troubleshooting
- Date parsing errors: ensure dates are in ISO format or specify date format in config.
- Encoding issues: pass encoding argument or convert files to UTF-8.
- Large files: consider streaming or increasing available memory; use chunked processing for very large exports.

License
Specify a license for the project (e.g., MIT). If none exists, add a LICENSE file.

Contact
For questions or support, open an issue on this repository or contact the maintainer listed in the repository profile.

Acknowledgements
- Built with pandas for fast tabular processing.
- Inspired by common AP reconciliation workflows.
