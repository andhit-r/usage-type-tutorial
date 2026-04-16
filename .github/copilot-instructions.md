---
applyTo: "**"
---

# Copilot Instructions — usage-type-tutorial

This project is a **MkDocs documentation site** explaining how `product.usage_type`
is used across SSI HR Expense modules (Odoo 14). It is intended to be deployed on
GitHub Pages via the `mkdocs gh-deploy` workflow.

---

## Project Purpose

Document the `product.usage_type` mechanism and its integration with:

- `hr.reimbursement` + `hr.reimbursement_line`
- `hr.cash_advance` + `hr.cash_advance_line`
- `hr.cash_advance_settlement` + `hr.cash_advance_settlement_line`

The source code being documented lives in:
- `/home/andhit-r/odoo14-development/odoo/custom/src/opnsynid-hr-expense/`
- `/home/andhit-r/odoo14-development/odoo/custom/src/ssi-mixin/`
- `/home/andhit-r/odoo14-development/odoo/custom/src/ssi-product-attribute/`

---

## Documentation Structure

```
docs/
  index.md                          ← Landing page + architecture overview
  concept/
    product-usage-type.md           ← product.usage_type model
    mixin-product-line-account.md   ← mixin.product_line_account
    account-resolution.md           ← 4-level account resolution hierarchy
  use-cases/
    index.md                        ← Overview table for all 3 objects
    reimbursement.md                ← hr.reimbursement use case
    cash-advance.md                 ← hr.cash_advance use case
    cash-advance-settlement.md      ← hr.cash_advance_settlement use case
  reference/
    comparison.md                   ← Side-by-side comparison table
mkdocs.yml                          ← MkDocs config (Material theme)
requirements.txt                    ← mkdocs + mkdocs-material
.github/workflows/deploy.yml        ← GitHub Actions for gh-pages deploy
```

---

## Writing Guidelines

- **Language**: Indonesian (Bahasa Indonesia) for all documentation text.
- **Code examples**: Use Python as shown in the source files. Do not fabricate field
  names or method signatures — always verify against the actual source files.
- **Diagrams**: Use Mermaid (`mermaid` fenced code blocks). MkDocs Material supports
  them via `pymdownx.superfences`.
- **Admonitions**: Use `!!! note`, `!!! tip`, `!!! info`, `!!! warning` where relevant.

## Key Concepts to Always Remember

1. `product.usage_type` has a `code` field (from `mixin.master_data`) used as
   `usage_code` in account resolution.
2. Account resolution priority: Product → Product Template → Product Category → Usage Type.
3. `hr.expense_type.default_product_usage_id` drives the automatic selection of
   `usage_id` in line models via `onchange_line_usage_id`.
4. In `hr.cash_advance_line`, `account_id` is **informational only** — the header's
   `cash_advance_account_id` / `payable_account_id` are used for the actual journal.
5. In `hr.cash_advance_settlement_line`, `account_id` **is used** in the journal entry
   to record actual expenses.

## MkDocs Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Serve locally (auto-reload)
mkdocs serve

# Build static site
mkdocs build

# Deploy to GitHub Pages
mkdocs gh-deploy
```
