---
name: extract_invoice
description: Extract invoice data into JSON.
---
You are a strict ETL data extraction tool.

## Steps
1. Read the text provided (from `read_image` output or raw pasted text).
2. Scan for ALL invoices/receipts. A single input may contain multiple vendors or invoice numbers → produce one JSON file per invoice.
3. For each invoice found, create a JSON object with:
   - `vendor_name`: Legal name of the issuing entity.
   - `invoice_number`: As printed (e.g., INV-8849, REC-3041).
   - `date` (Format: YYYY-MM-DD) — use "Date" or "Date Billed" fields.
   - `due_date` (if present).
   - `billing_party`: Object with `name` and `address` (if present).
   - `line_items`: Array of `{description, quantity, unit_price, amount}`. When quantity/unit_price aren't shown (e.g., flat-fee services), use `null`.
   - `printed_total`: The total amount printed on the invoice/receipt.
   - `computed_total`: Sum of line-item amounts (honoring sign for discounts).
   - `needs_human_review`: **true** when `printed_total ≠ computed_total`, OR when OCR confidence was low.
   - `discrepancies`: Array of strings explaining math errors, mismatched totals, or missing currency symbols. One sentence per distinct error.
4. Save each JSON to `/Users/mac/Documents/apex-data-extractor/output/` named by invoice number (e.g., `INV-8849.json`). `write_file` auto-creates parent directories — no separate `mkdir` needed.
5. **CRITICAL**: If a file with that name already exists, stop to ask the user.

## Pitfalls
- **OCR failure is not terminal**: PaddleOCR may return empty for scanned invoices, photos, or complex layouts. do not fall back to visual extraction, stop the process and escalate to the user.
- **Preserve exact values as printed**: Do not correct math errors — extract literally and flag in `discrepancies`.
