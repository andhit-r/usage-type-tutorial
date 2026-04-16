# SSI Product Usage Type — Tutorial

Selamat datang di dokumentasi **Product Usage Type** pada ekosistem modul HR Expense
SSI (PT. Simetri Sinergi Indonesia / OpenSynergy Indonesia) untuk Odoo 14.

---

## Apa yang Akan Dipelajari?

Dokumentasi ini menjelaskan bagaimana **`product.usage_type`** (Product Usage Type)
berfungsi sebagai jembatan antara **produk** dan **akun akuntansi** pada transaksi
pengeluaran karyawan, mencakup:

| Topik | Deskripsi |
|---|---|
| [Product Usage Type](concept/product-usage-type.md) | Model master data `product.usage_type` |
| [Mixin Product Line Account](concept/mixin-product-line-account.md) | Mixin yang mengintegrasikan Usage ke baris transaksi |
| [Alur Resolusi Akun](concept/account-resolution.md) | Hierarki pencarian akun berdasarkan usage code |
| [Reimbursement](use-cases/reimbursement.md) | Penerapan pada `hr.reimbursement` |
| [Cash Advance](use-cases/cash-advance.md) | Penerapan pada `hr.cash_advance` |
| [Cash Advance Settlement](use-cases/cash-advance-settlement.md) | Penerapan pada `hr.cash_advance_settlement` |

---

## Gambaran Umum Arsitektur

```mermaid
graph TD
    A[product.usage_type<br/><i>Master Data</i>] -->|usage_code| B[product._get_product_account]
    B --> C[account.account]

    D[hr.expense_type<br/><i>Expense Type Config</i>] -->|allowed_product_usage_ids| A
    D -->|default_product_usage_id| A

    E[Baris Transaksi<br/>mixin.product_line_account] -->|usage_id| A
    E -->|account_id| C

    F[hr.reimbursement_line] -->|inherit| E
    G[hr.cash_advance_line] -->|inherit| E
    H[hr.cash_advance_settlement_line] -->|inherit| E
```

---

## Konsep Inti

**Product Usage Type** menjawab pertanyaan:

> _"Produk X digunakan untuk keperluan apa, sehingga harus dicatat ke akun mana?"_

Satu produk yang sama (misalnya "Tiket Pesawat") dapat diposting ke akun berbeda
bergantung pada **konteks penggunaannya**:

- Perjalanan dinas → akun **Biaya Perjalanan Dinas**
- Training → akun **Biaya Pelatihan**
- Proyek klien → akun **WIP Project**

Usage Type-lah yang membedakan konteks tersebut.
