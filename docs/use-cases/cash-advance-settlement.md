# Cash Advance Settlement — Penggunaan Usage Type

**Model:** `hr.cash_advance_settlement` + `hr.cash_advance_settlement_line`

---

## Bagaimana `usage_id` Diisi

Mekanisme pengisian `usage_id` pada baris settlement sama dengan cash advance dan
reimbursement:

```python
# hr.cash_advance_settlement_line
@api.onchange("product_id")
def onchange_line_usage_id(self):
    self.usage_id = False
    if self.product_id:
        self.usage_id = self.type_id.default_product_usage_id.id
```

`type_id` di baris adalah `related` ke `cash_advance_settlement_id.type_id`.

---

## Bagaimana `account_id` Ditentukan

Resolusi akun berjalan otomatis setelah `usage_id` terisi, melalui mixin yang sama:

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

## `account_id` Baris Dipakai di Journal Entry

!!! tip "Perbedaan kritis dengan Cash Advance"
    Di settlement, `account_id` dari setiap baris **benar-benar dipakai** sebagai
    akun debit pada jurnal pengeluaran aktual — berbeda dengan `hr.cash_advance_line`
    yang hanya informatif.

```python
# hr.cash_advance_settlement_line
def _prepare_create_expense_line_data(self, move):
    return {
        "account_id": self.account_id.id,   # ← hasil resolusi usage
        "debit": amount,
        "credit": 0.0,
        ...
    }
```

| Debit | Kredit |
|---|---|
| `line.account_id` per baris (akun belanja aktual) | `cash_advance_move_line_id` (clearing uang muka) |
