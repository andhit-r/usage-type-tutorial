# SSI Product Usage Type — Tutorial

Dokumentasi penggunaan **`product.usage_type`** pada modul HR Expense SSI (Odoo 14),
mencakup integrasi dengan Reimbursement, Cash Advance, dan Cash Advance Settlement.

Dibangun dengan [MkDocs Material](https://squidfunk.github.io/mkdocs-material/) dan
di-deploy ke GitHub Pages.

---

## Topik Dokumentasi

| Halaman | Keterangan |
|---|---|
| Konsep: Product Usage Type | Model `product.usage_type` — fields, methods, konfigurasi |
| Konsep: Mixin Product Line Account | Mixin yang mengintegrasikan usage ke baris transaksi |
| Konsep: Alur Resolusi Akun | Hierarki 4-level pencarian akun berdasarkan `usage_code` |
| Use Case: Reimbursement | Penerapan pada `hr.reimbursement` + `hr.reimbursement_line` |
| Use Case: Cash Advance | Penerapan pada `hr.cash_advance` + `hr.cash_advance_line` |
| Use Case: Cash Advance Settlement | Penerapan pada `hr.cash_advance_settlement` + line |
| Referensi: Perbandingan | Tabel perbandingan ketiga objek secara berdampingan |

---

## Source Code

Kode sumber yang didokumentasikan berasal dari:

- [`opnsynid-hr-expense`](https://github.com/open-synergy/opnsynid-hr-expense) — modul HR Expense
- [`ssi-mixin`](https://github.com/open-synergy/ssi-mixin) — `mixin.product_line_account`
- [`ssi-product-attribute`](https://github.com/open-synergy/ssi-product-attribute) — `product.usage_type`

---

## Menjalankan Secara Lokal

```bash
# Buat virtual environment
python3 -m venv .venv
source .venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Serve dengan auto-reload
mkdocs serve
```

Buka <http://127.0.0.1:8000> di browser.

## Deploy ke GitHub Pages

```bash
mkdocs gh-deploy
```

Atau push ke branch `main` — GitHub Actions akan men-deploy otomatis.

---

## Lisensi

AGPL-3.0 — PT. Simetri Sinergi Indonesia / OpenSynergy Indonesia
