# Product Usage Type

**Model:** `product.usage_type`
**Modul:** `ssi_product_usage_account_type`
**Inherit:** `mixin.master_data`

---

## Tujuan

`product.usage_type` adalah master data yang **mendefinisikan konteks penggunaan sebuah
produk** dan menghubungkannya dengan akun akuntansi serta pajak yang sesuai.

Tanpa Usage Type, satu produk hanya memiliki satu akun. Dengan Usage Type, produk yang
sama dapat diposting ke akun berbeda bergantung pada _untuk apa_ produk itu digunakan.

---

## Definisi Model

```python
class ProductUsage(models.Model):
    _name = "product.usage_type"
    _inherit = ["mixin.master_data"]
    _description = "Product Usage Type"

    selection_method = fields.Selection(
        selection=[("fixed", "Fixed"), ("python", "Python Code")],
        required=True,
        default="fixed",
    )
    account_id = fields.Many2one(
        comodel_name="account.account",
        required=True,
        ondelete="restrict",
        company_dependent=True,          # (1)
    )
    tax_selection_method = fields.Selection(
        selection=[("fixed", "Fixed"), ("python", "Python Code")],
        required=True,
        default="fixed",
    )
    tax_ids = fields.Many2many(
        comodel_name="account.tax",
        relation="rel_usage_type_2_tax",
    )
```

1. `company_dependent=True` berarti nilai `account_id` bisa berbeda per perusahaan
   dalam environment multi-company.

---

## Fields Utama

| Field | Tipe | Keterangan |
|---|---|---|
| `name` | Char | Nama usage type (dari `mixin.master_data`) |
| `code` | Char | Kode unik; digunakan sebagai `usage_code` saat resolusi akun |
| `selection_method` | Selection | `fixed` (langsung dari `account_id`) atau `python` (evaluasi kode Python) |
| `account_id` | Many2one | Akun fallback terakhir; `company_dependent` |
| `tax_selection_method` | Selection | `fixed` (dari `tax_ids`) atau `python` |
| `tax_ids` | Many2many | Pajak fallback terakhir |

---

## Methods

### `_get_account(local_dict=False)`

Mengembalikan `account.account` berdasarkan `selection_method`.

- Jika `fixed` → langsung `self.account_id`
- Jika `python` → evaluasi `python_code` dengan `local_dict`

### `_get_tax(local_dict=False)`

Mengembalikan list ID `account.tax`.

- Jika `fixed` → `self.tax_ids.ids`
- Jika `python` → evaluasi `python_code` dengan `local_dict`

---

## Hubungan dengan hr.expense_type

Ketika mengkonfigurasi `hr.expense_type`, terdapat field:

```python
allowed_product_usage_ids = fields.Many2many(
    comodel_name="product.usage_type",
    relation="rel_expense_type_2_product_usage",
)
default_product_usage_id = fields.Many2one(
    comodel_name="product.usage_type",
)
```

- **`allowed_product_usage_ids`**: membatasi pilihan usage yang tersedia di baris
  transaksi untuk expense type ini.
- **`default_product_usage_id`**: usage yang otomatis terisi saat user memilih produk
  pada baris transaksi.

---

## Contoh Konfigurasi

```
Usage Type: "Biaya Perjalanan Dinas"
  code             : travel
  selection_method : fixed
  account_id       : 6-1001 (Biaya Perjalanan)
  tax_ids          : (kosong)

Usage Type: "Biaya Hotel"
  code             : hotel
  selection_method : fixed
  account_id       : 6-1002 (Biaya Akomodasi)
  tax_ids          : PPN 11%
```

Expense Type "Perjalanan Dinas" kemudian dikonfigurasi dengan:
```
allowed_product_usage_ids : [travel, hotel]
default_product_usage_id  : travel
```

Sehingga setiap kali karyawan menginput baris expense dengan produk apapun dalam
expense type ini, usage default yang terisi adalah "travel", dan akun yang dipakai
adalah akun 6-1001.
