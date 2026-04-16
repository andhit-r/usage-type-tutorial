# Cash Advance — Penggunaan Usage Type

**Model:** `hr.cash_advance` + `hr.cash_advance_line`

---

## Bagaimana `usage_id` Diisi

Mekanisme pengisian `usage_id` pada baris cash advance **identik** dengan reimbursement:

```python
# hr.cash_advance_line
@api.onchange("product_id")
def onchange_line_usage_id(self):
    self.usage_id = False
    if self.product_id:
        self.usage_id = self.type_id.default_product_usage_id.id
```

`type_id` di baris adalah `related` ke `cash_advance_id.type_id`, sehingga
`default_product_usage_id` dibaca dari expense type yang dipilih di header.

---

## Bagaimana `account_id` Ditentukan

Setelah `usage_id` terisi, mixin secara otomatis menjalankan resolusi akun:

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

---

## `account_id` Baris Bersifat Informatif

!!! warning "Penting"
    Berbeda dengan reimbursement, `account_id` pada `hr.cash_advance_line` **tidak
    dimasukkan ke journal entry** saat cash advance disetujui.

Jurnal yang dibuat saat `action_open()` hanya menggunakan akun yang ada di **header**:

| Debit | Kredit |
|---|---|
| `cash_advance_account_id` (piutang uang muka ke karyawan) | `payable_account_id` (hutang ke karyawan) |

`account_id` dari baris berfungsi sebagai **rencana akun belanja** — mencerminkan ke
akun mana pengeluaran nyata akan dicatat nanti pada saat settlement dibuat.
