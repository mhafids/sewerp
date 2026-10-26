# Standar Database & Administrasi Data PostgreSQL SewERP

Dokumen ini memuat panduan perancangan skema, konvensi penamaan tabel/kolom, strategi pengindeksan, manajemen migrasi, dan pemeliharaan PostgreSQL untuk SewERP.

---

## Rekomendasi Skills untuk Agent (Recommended Agent Skills)

Ketika agen bertugas merancang DDL skema, menulis file migrasi goose, menyusun query sqlc, atau mengoptimalkan indeks PostgreSQL:
- `erd-api-contract`: Menghasilkan diagram relasi entitas (Mermaid), primary key time-ordered UUIDv7, skema multi-tenancy `tenant_id`, tipe moneter `BIGINT`, dan persentase basis points `rate_bps`.
- `spec-kit`: Memastikan kepatuhan terhadap aturan penamaan snake_case, soft-delete triggers, dan checklist kolom audit trail standar.
- `codegraph`: Melacak berkas SQL mentah, query sqlc yang dihasilkan, serta dependensi pemanggilan repository di backend Go.

---

## 1. Mesin Database & Perangkat Pendukung

- **Database RDBMS**: [PostgreSQL](https://www.postgresql.org) v16+.
- **Alat Migrasi**: [goose](https://github.com/pressly/goose) (format migrasi SQL native bertanggal/bernomor).
- **Generator Kode**: [sqlc](https://sqlc.dev) (menghasilkan query Go berkecepatan tinggi tanpa refleksi ORM).
- **Karakter Set & Timezone**: UTF-8 dengan default timezone server `UTC`. Penyimpanan waktu menggunakan `TIMESTAMPTZ`.

---

## 2. Konvensi Penamaan (Naming Conventions)

| Unsur Skema | Konvensi | Contoh Baku | Larangan |
| :--- | :--- | :--- | :--- |
| **Nama Tabel** | `snake_case`, jamak (*plural*) | `sales_orders`, `order_items`, `fabric_rolls` | `SalesOrder`, `tbl_order` |
| **Primary Key** | `id` bertipe `UUID` (UUIDv7) | `id UUID PRIMARY KEY DEFAULT gen_random_uuid()` | `order_id`, `id_order` |
| **Foreign Key** | Nama entitas tunggal + `_id` | `customer_id`, `sales_order_id`, `tenant_id` | `fk_customer`, `id_pelanggan` |
| **Kolom Boolean** | Awalan `is_` atau `has_` | `is_active`, `is_paid`, `has_surcharge` | `status_aktif`, `flag_bayar` |
| **Kolom Waktu** | Akhiran `_at` | `created_at`, `approved_at`, `shipped_at` | `tgl_buat`, `order_time` |
| **Kolom Nilai Uang** | `BIGINT`, tanpa koma | `unit_price`, `subtotal_amount`, `grand_total` | bertipe `FLOAT`/`NUMERIC(10,2)` |
| **Kolom Persentase** | `_bps` (Basis Points, `INT`) | `discount_bps`, `tax_bps`, `wastage_margin_bps` | bertipe desimal koma |

---

## 3. Kolom Standar & Audit Trail

Setiap tabel master dan transaksional wajib memiliki blok kolom audit trail berikut:

```sql
tenant_id    UUID NOT NULL,
created_at   TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP,
updated_at   TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP,
created_by   UUID NULL,
updated_by   UUID NULL,
deleted_at   TIMESTAMPTZ NULL
```

- **Soft Delete**: Data pesanan, master pelanggan, dan inventaris tidak dihapus secara permanen (`HARD DELETE`), melainkan diisi tanggal `deleted_at`.
- **Trigger `updated_at`**: Setiap tabel memiliki trigger fungsi PostgreSQL otomatis `set_updated_at()` saat terjadi pembaruan baris.

---

## 4. Strategi Pengindeksan (Indexing Strategy)

Untuk menjamin latensi query sub-100ms dalam skala ratusan ribu transaksi pesanan konveksi:

1. **Compound Multi-Tenant Index**:
   Setiap pencarian list order atau transaksi wajib menggunakan indeks gabungan tenant:
   ```sql
   CREATE INDEX idx_sales_orders_tenant_created 
   ON sales_orders (tenant_id, created_at DESC);
   ```

2. **Partial Index untuk Data Aktif (Soft Delete Filtering)**:
   Mencegah pembengkakan ukuran index dengan hanya mengindeks record yang belum dihapus:
   ```sql
   CREATE INDEX idx_customers_active 
   ON customers (tenant_id, name) 
   WHERE deleted_at IS NULL;
   ```

3. **Composite Unique Index per Tenant**:
   Nomor SPK atau nomor Quotation harus unik hanya di dalam cakupan tenant yang sama:
   ```sql
   CREATE UNIQUE INDEX uq_sales_orders_tenant_number 
   ON sales_orders (tenant_id, order_number) 
   WHERE deleted_at IS NULL;
   ```

---

## 5. Standar File Migrasi Goose

File migrasi disimpan di direktori `backend/internal/common/db/migrations/` dengan format:
`YYYYMMDDHHMMSS_nama_aksi.sql`

Contoh: `20261022000001_create_production_tables.sql`

Struktur berkas wajib memisahkan blok `-- +goose Up` dan `-- +goose Down`:
```sql
-- +goose Up
-- +goose StatementBegin
CREATE TABLE production_work_orders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL,
    sales_order_id UUID NOT NULL REFERENCES sales_orders(id) ON DELETE RESTRICT,
    work_order_number VARCHAR(64) NOT NULL,
    stage VARCHAR(32) NOT NULL,
    total_cutting_pcs INT NOT NULL DEFAULT 0,
    created_at TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMPTZ NULL
);
-- +goose StatementEnd

-- +goose Down
-- +goose StatementBegin
DROP TABLE IF EXISTS production_work_orders;
-- +goose StatementEnd
```

---

## 6. Referensi & Dokumen Terkait

- 🏛️ Standar Arsitektur Sistem: [architecture.md](./architecture.md)
- ⚙️ Standar Rekayasa Backend: [backend.md](./backend.md)
- 🚀 Standar DevOps & Backup Database: [devops.md](./devops.md)
- 🧪 Pedoman QA & Verifikasi Data: [qa.md](./qa.md)
- 📦 Spesifikasi Manajemen Inventaris (Kain & Stok): [../specs/005a-fabric-inventory.md](../specs/005a-fabric-inventory.md)
- 📋 Standar Dokumentasi Requirement: [requirements_standard.md](./requirements_standard.md)
