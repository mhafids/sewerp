# Standar Rekayasa Backend Go SewERP

Dokumen ini memuat panduan implementasi, struktur direktori, konvensi kode, manajemen transaksi, penanganan error, validasi ketat (*strict validation*), serta persyaratan uji tolak ukur performa (*benchmark testing*) berbasis **Zero Overhead, Zero Memory Allocation, dan Zero Footprint** untuk SewERP.

---

## Rekomendasi Skills untuk Agent (Recommended Agent Skills)

Ketika agen bertugas merancang atau mengimplementasikan modul backend, database migration, API handler, atau worker background di Go, aktifkan dan kombinasikan skills berikut:
- `erd-api-contract`: Merancang migrasi skema PostgreSQL (goose), kueri sqlc type-safe, entitas domain dengan presisi moneter `BIGINT` & `rate_bps`, serta kontrak endpoint RESTful berbasis DTO beranotasi validator kustom.
- `spec-kit`: Menjaga konsistensi arsitektur Clean Architecture, spesifikasi interface domain, dan verifikasi checklist batasan bounded context.
- `codegraph`: Menjelajahi kode sumber verbatim, menelusuri pemanggilan fungsi antar layer (`domain` -> `usecase` -> `repository` -> `delivery`), dan memverifikasi batas dependensi tanpa loop baca manual.
- `go-vulncheck`: Memeriksa potensi kerentanan keamanan (CVE) pada modul pihak ketiga di Go (`go.mod`) sebelum deployment.

---

## 1. Teknologi Backend Utama

- **Bahasa**: Go 1.23+ (menggunakan native slicing, context, dan generic).
- **Web Framework**: [Gin Gonic](https://gin-gonic.com) (routing berkinerja tinggi, zero allocation middleware).
- **Database Driver**: [pgx/v5](https://github.com/jackc/pgx) dengan pool koneksi terkelola (`pgxpool`).
- **Data Access Engine**: [sqlc](https://sqlc.dev) (kompilasi query SQL mentah menjadi Go code yang type-safe tanpa overhead runtime reflection ORM).
- **Database Migration**: [goose](https://github.com/pressly/goose) (migrasi SQL murni yang dapat di-rollback).
- **Background Worker**: [River](https://github.com/riverqueue/river) (antrean job transaksional native PostgreSQL).
- **Validasi Data**: `go-playground/validator/v10` dengan custom domain validation tags.
- **Structured Logging**: [Zerolog](https://github.com/rs/zerolog) dengan format JSON terstruktur untuk observabilitas.

---

## 2. Struktur Layer Clean Architecture

Setiap bounded context mengikuti struktur layer terisolasi di bawah `backend/internal/{context_name}`:

```text
backend/internal/oms/
├── domain/                  # Entitas murni, Value Objects, Domain Errors, Interface Repository
│   ├── order.go
│   ├── quotation.go
│   ├── errors.go
│   └── repository.go
├── usecase/                 # Orchestration logika bisnis, transaksi DB, pemanggilan service
│   ├── create_order.go
│   ├── approve_quotation.go
│   └── calculate_pricing.go
├── repository/              # Implementasi database akses menggunakan sqlc + pgx/v5
│   ├── order_repository.go
│   └── sqlc/                # Berkas autogen dari sqlc (*.sql.go)
└── delivery/
    └── http/                # Controller Gin, DTO request/response, mapping HTTP status
        ├── handler.go
        ├── request.go
        └── response.go
```

### Aturan Ketergantungan (Dependency Rule)
- Layer `domain` tidak boleh mengimpor package dari layer `usecase`, `repository`, maupun `delivery`.
- Layer `usecase` hanya bergantung pada interface di layer `domain`.
- Layer `delivery/http` hanya bertugas mem-parsing HTTP request DTO, memvalidasi payload, memanggil `usecase`, dan membungkus hasil dalam standar envelope response.

---

## 3. Validasi Ketat (Strict Input Validation)

Semua input yang masuk ke sistem SewERP wajib melewati validasi ultra-ketat tanpa toleransi:

1. **Larangan Loose Types**: Dilarang keras menggunakan tipe `interface{}` atau `any` pada DTO request atau struct entitas bisnis. Setiap field wajib memiliki tipe data konkret.
2. **Validasi Format Keras**:
   - Kolom ID: Wajib validasi UUIDv7 menggunakan regex atau custom validator `uuid_v7`.
   - Nomor WhatsApp: Wajib mematuhi format standar E.164 (`+628...` atau `08...`) dengan panjang 10–15 digit.
   - Angka Nominal: Wajib integer positif bertipe `int64` (BIGINT), dilarang nilai negatif kecuali untuk potongan diskon yang didefinisikan secara terpisah.
3. **Larangan Enum Hardcoding**:
   - Dilarang keras menggunakan tag seperti `binding:"oneof=draft pending approved"`.
   - Seluruh enum wajib divalidasi menggunakan custom validator yang terdaftar pada runtime validator Go dan mencocokkan ke konstanta domain:
     ```go
     type CreateOrderRequest struct {
         CustomerID string `json:"customer_id" binding:"required,uuid_v7"`
         ArticleType string `json:"article_type" binding:"required,article_type_enum"`
         TotalPcs    int64  `json:"total_pcs" binding:"required,min=12,max=100000"`
     }
     ```
4. **Sanitasi String**: Seluruh input teks (nama pelanggan, catatan) wajib melewati pembersihan spasi berlebih (*trimming*) dan *HTML escaping* untuk mencegah XSS.

---

## 4. Standar Uji Tolak Ukur (Benchmark Requirements): Zero Overhead, Zero Alloc, Zero Footprint

Untuk menjamin kinerja backend berskala industri manufaktur dengan jutaan data transaksi tanpa *Garbage Collection (GC) pauses*:

### A. Target Kinerja Alur Kritis (Critical Hot Paths)
Fungsi-fungsi matematis dan parsing wajib memenuhi kriteria **`0 B/op` dan `0 allocs/op`**:
1. **Aritmatika Finansial (`types.Money`)**: Penjumlahan subtotal, kalkulasi diskon basis points (`rate_bps`), dan penambahan PPN.
2. **Kalkulasi Toleransi Susut Kain (`CalculateWastageMargin`)**: Formula penambahan 3% - 5% kain mentah.
3. **Validasi & Penjumlahan Matriks Ukuran Garmen**: Parsing array ukuran (S, M, L, XL, XXL) menjadi total pcs pesanan.
4. **Pengecekan Anti-Overclaim Borongan**: Validasi akumulasi klaim penjahit vs kuantiti potong.

### B. Struktur Uji Benchmark (`testing.B`)
Setiap modul kalkulasi domain wajib menyertakan berkas `*_bench_test.go` dengan pemanggilan `b.ReportAllocs()`:
```go
func BenchmarkCalculatePricingWithSurcharges(b *testing.B) {
    basePrice := types.Money(48000)
    sizes := []OrderItemSize{
        {SizeCode: "L", Quantity: 50, SurchargePerPcs: 0},
        {SizeCode: "XXL", Quantity: 10, SurchargePerPcs: 5000},
    }

    b.ResetTimer()
    b.ReportAllocs() // Wajib: Melaporkan alokasi memori per operasi

    for i := 0; i < b.N; i++ {
        _ = CalculateTotalWithSurcharges(basePrice, sizes)
    }
}
```
**Ambang Kelulusan CI/CD**:
```text
BenchmarkCalculatePricingWithSurcharges-8   50000000   22.4 ns/op   0 B/op   0 allocs/op
PASS
```

### C. Escape Analysis & Zero Memory Footprint
- **Analisis Alokasi Heap**: Developer wajib memverifikasi bahwa variabel domain tidak "kabur" ke heap (*escape to heap*) menggunakan perintah kompilasi:
  ```bash
  go build -gcflags="-m -m" ./...
  ```
- **Zero Memory Leaks (Zero Footprint)**:
  - Setiap goroutine yang dijalankan wajib memiliki pembatalan berbasis `context.WithTimeout` atau `context.WithCancel`.
  - Pool objek memori (`sync.Pool`) wajib digunakan untuk buffer serialisasi JSON atau byte slice besar.
  - Profiling memori reguler menggunakan `go tool pprof` untuk memastikan tidak ada alokasi memori yang tertahan (*lingering memory allocations*).

---

## 5. Standar Format Respons REST & Envelope

### Respons Sukses (`ApiResponse<T>`)
```json
{
  "status": "success",
  "data": {
    "id": "01925b6a-8451-7ef1-953e-51c045b0a701",
    "order_number": "PO-202610-0012",
    "customer_name": "PT Sumber Busana",
    "total_amount": 18500000,
    "status": "in_production"
  },
  "meta": {
    "page": 1,
    "per_page": 20,
    "total_items": 1,
    "total_pages": 1
  }
}
```

### Respons Gagal (RFC 7807 Problem Details)
```json
{
  "type": "https://api.sewerp.id/errors/insufficient-stock",
  "title": "Bahan Baku Tidak Cukup",
  "status": 422,
  "detail": "Kain Cotton Combed 30s Hitam Reaktif hanya tersisa 12 roll, dibutuhkan 15 roll (termasuk wastage margin 4%).",
  "instance": "/api/v1/oms/orders/01925b6a-8451-7ef1-953e-51c045b0a701/approve",
  "invalid_params": [
    {
      "field": "fabric_id",
      "reason": "Stok tidak mencukupi alokasi minimal SPK"
    }
  ]
}
```

---

## 6. Konkurensi & Penguncian Transaksi Database

Untuk mencegah *race condition* pada operasional penting konveksi:
1. **Alokasi Bahan Baku**: Saat SPK disetujui, query stok kain wajib menggunakan penguncian baris eksplisit:
   ```sql
   SELECT id, available_yards FROM fabric_stocks 
   WHERE id = $1 AND tenant_id = $2 
   FOR UPDATE;
   ```
2. **Penomoran Urut SPK & Quotation**: Menggunakan sequence terisolasi per tenant atau transactional counter table untuk menghindari nomor SPK yang melompat (*gapless*) atau duplikat pada order serentak.

---

## 7. Referensi & Dokumen Terkait

- 🏛️ Standar Arsitektur Global: [architecture.md](./architecture.md)
- 🗄️ Standar Database & Migrasi: [dbadmin.md](./dbadmin.md)
- 🧪 Pedoman Pengujian Backend & QA: [qa.md](./qa.md)
- 📋 Aturan Bisnis Alur Konveksi: [business_requirements.md](./business_requirements.md)
- 🔧 Kebutuhan Teknis Non-Fungsional: [technical_requirements.md](./technical_requirements.md)
- 📄 Spesifikasi Sales & Order (OMS): [../specs/003a-sales-order-core.md](../specs/003a-sales-order-core.md)
- 📋 Standar Dokumentasi Requirement: [requirements_standard.md](./requirements_standard.md)
