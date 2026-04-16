# Reimbursement — Penggunaan Usage Type

**Model:** `hr.reimbursement` + `hr.reimbursement_line`

---

## Bagaimana `usage_id` Diisi

Saat user memilih `product_id` pada baris reimbursement, `usage_id` otomatis diisi
dari konfigurasi default di `hr.expense_type` header:

```python
# hr.reimbursement_line
@api.onchange("product_id")
def onchange_line_usage_id(self):
    self.usage_id = False
    if self.product_id:
        self.usage_id = self.type_id.default_product_usage_id.id
```

`type_id` di baris adalah field `related` ke `reimbursement_id.type_id`, sehingga
nilai `default_product_usage_id` dibaca langsung dari expense type yang dipilih di
header.

!!! info "Aturan pilihan usage"
    Pilihan `usage_id` yang tersedia di baris dibatasi oleh
    `type_id.allowed_product_usage_ids`. User hanya bisa mengganti ke usage lain
    yang ada dalam daftar ini.

---

## Bagaimana `account_id` Ditentukan

Setelah `usage_id` terisi, mixin `onchange_account_id` memanggil resolusi akun:

```python
# mixin.product_line_account
@api.onchange("usage_id", "product_id")
def onchange_account_id(self):
    self.account_id = False
    if self.product_id and self.usage_id:
        self.account_id = self.product_id._get_product_account(
            usage_code=self.usage_id.code
        )
```

Resolusi mengikuti hierarki 4 level (lihat [Alur Resolusi Akun](../concept/account-resolution.md)).

---

## Penggunaan `account_id` di Journal Entry

`account_id` dari setiap baris **langsung dipakai** sebagai akun debit pengeluaran
pada `account.move.line` saat reimbursement selesai disetujui:

```python
# hr.reimbursement_line
def _prepare_expense_move_lines(self):
    return {
        "account_id": self.account_id.id,   # ← hasil resolusi usage
        "debit": debit,
        "credit": credit,
        ...
    }
```

| Debit | Kredit |
|---|---|
| `line.account_id` per baris (akun belanja) | `header.account_id` (akun payable karyawan) |
