# Ringkasan Perbandingan

Halaman ini menyajikan perbandingan komprehensif penggunaan `product.usage_type`
di tiga objek transaksi.

---

## Perbandingan Header

| Aspek | `hr.reimbursement` | `hr.cash_advance` | `hr.cash_advance_settlement` |
|---|---|---|---|
| Modul | `ssi_hr_reimbursement` | `ssi_hr_cash_advance` | `ssi_hr_cash_advance` |
| Alur status | draft→confirm→open→done | draft→confirm→open→done | draft→confirm→done |
| Punya `type_id`? | ✅ | ✅ | ✅ |
| `allowed_product_usage_ids` | Via `type_id` | Via `type_id` | Via `type_id` |
| Akun di header | `account_id` (payable) | `payable_account_id` + `cash_advance_account_id` | Via `cash_advance_id` |
| Baris model | `hr.reimbursement_line` | `hr.cash_advance_line` | `hr.cash_advance_settlement_line` |

---

## Perbandingan Baris

| Aspek | `hr.reimbursement_line` | `hr.cash_advance_line` | `hr.cash_advance_settlement_line` |
|---|---|---|---|
| Inherit | `mixin.product_line_account` | `mixin.product_line_account` | `mixin.product_line_account` |
| Parent FK | `reimbursement_id` | `cash_advance_id` | `cash_advance_settlement_id` |
| `type_id` (related) | `reimbursement_id.type_id` | `cash_advance_id.type_id` | `cash_advance_settlement_id.type_id` |
| `usage_id` default dari | `type_id.default_product_usage_id` | `type_id.default_product_usage_id` | `type_id.default_product_usage_id` |
| `account_id` dipakai di jurnal | ✅ Ya | ❌ Hanya informatif | ✅ Ya |
| Field `date_expense` | ✅ | ✅ | ✅ |

---

## Perbandingan Fungsi Akun di Baris

```mermaid
graph TD
    subgraph Reimbursement
        R1[account_id dari usage] -->|dipakai di| R2[account.move.line<br/>debit belanja]
    end

    subgraph Cash Advance
        C1[account_id dari usage] -->|hanya informatif| C2[rencana akun belanja]
        C3[cash_advance_account_id] -->|dipakai di| C4[account.move.line<br/>debit uang muka]
        C5[payable_account_id] -->|dipakai di| C6[account.move.line<br/>kredit hutang karyawan]
    end

    subgraph Settlement
        S1[account_id dari usage] -->|dipakai di| S2[account.move.line<br/>debit belanja aktual]
        S3[cash_advance_move_line_id] -->|dipakai di| S4[account.move.line<br/>kredit clearing uang muka]
    end
```

---

## Alur Lengkap: Cash Advance → Settlement

```mermaid
flowchart LR
    CA[hr.cash_advance<br/>state: open] -->|direferens oleh| CAS[hr.cash_advance_settlement]
    CA -->|jurnal: Debit CA Account<br/>Kredit Payable| JE1[account.move<br/>Jurnal Uang Muka]
    CAS -->|jurnal: Debit line.account_id<br/>Kredit CA Move Line| JE2[account.move<br/>Jurnal Settlement]
    JE1 <-->|rekonsiliasi| JE2
    JE2 -->|settled = True| CA
```

---

## Rangkuman Konfigurasi yang Diperlukan

Untuk menggunakan fitur ini dengan benar, pastikan konfigurasi berikut sudah ada:

1. **`product.usage_type`** — buat usage type dengan kode unik dan `account_id`
2. **Produk** — opsional: konfigurasi `product.account` untuk override akun per produk
3. **`hr.expense_type`**:
   - Set `allowed_product_usage_ids` → usage yang boleh dipilih
   - Set `default_product_usage_id` → usage yang otomatis terisi
4. **Transaksi** — pilih `type_id` yang benar di header, lalu input baris produk
