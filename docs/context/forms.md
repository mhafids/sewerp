# Standar Formulir, Validasi Ketat & Penanganan Matriks Ukuran SewERP

Dokumen ini memuat arsitektur formulir, validasi skema ketat frontend (*Strict Zod*) & backend (*Go Validator*), penanganan matriks ukuran pakaian dinamis bebas alokasi memori (*zero memory allocations*), dan pemformatan mata uang untuk SewERP.

---

## Rekomendasi Skills untuk Agent (Recommended Agent Skills)

Ketika agen bertugas membangun formulir interaktif, validasi skema Zod `.strict()`, masking mata uang Rupiah, atau matriks ukuran dinamis:
- `ui-ux-pro-max`: Merancang tata letak formulir padat ergonomis, navigasi Tab/Enter horizontal pada sel matriks, dan penempatan tombol pintasan kuantiti.
- `memory-leak-debugging`: Menganalisis stabilitas mutasi path pada SolidJS `createStore` untuk memastikan ketiadaan garbage collection thrashing pada pengetikan cepat.
- `generative_ui`: Menghasilkan pratinjau komponen formulir interaktif dan visualisasi spreadsheet matriks ukuran secara instan.

---

## 1. Pustaka & Arsitektur Formulir

- **Frontend**: [modular-forms](https://modularforms.dev) dipadukan dengan [zod](https://zod.dev) dengan mode `.strict()` untuk validasi skema deklaratif, tipe data ketat, dan performa render instan tanpa lag saat mengetik.
- **Backend**: `go-playground/validator/v10` pada DTO request Gin-Gonic dengan pemetaan pesan error terstandar RFC 7807 dan custom domain validators.

---

## 2. Penanganan Matriks Ukuran Garmen Bebas Overhead (Zero Overhead Matrix)

Pesanan konveksi pakaian memiliki struktur data unik: satu artikel pakaian (misal: "Kaos Polo Bordir") dapat terdiri dari beberapa varian warna kain, dan setiap varian warna memiliki rincian jumlah per ukuran:

```text
┌─────────────────┬────┬────┬────┬────┬─────┬───────┐
│ Varian Warna    │ S  │ M  │ L  │ XL │ XXL │ Total │
├─────────────────┼────┼────┼────┼────┼─────┼───────┤
│ Hitam Reaktif   │ 12 │ 24 │ 36 │ 12 │  6  │  90   │
│ Navy Blue       │  6 │ 12 │ 24 │ 12 │  0  │  54   │
├─────────────────┴────┴────┴────┴────┴─────┼───────┤
│ TOTAL KESELURUHAN ARTIKEL                 │ 144   │
└───────────────────────────────────────────┴───────┘
```

### Aturan Teknis Matriks Input:
1. **Zero Garbage Collection Thrashing**: Dilarang meng-kloning atau me-recreate seluruh objek matriks (`{ ...matrix }`) pada setiap ketukan keyboard (*keystroke*). Gunakan mutasi path terfokus pada SolidJS `createStore`:
   ```typescript
   // Mutasi presisi tanpa alokasi objek baru
   setSizeMatrix(colorIndex, "sizes", sizeCode, (prev) => Math.max(0, newQty));
   ```
2. **Navigasi Tombol Tab & Enter**: Menekan tombol `Tab` atau `Enter` pada sel ukuran otomatis memindahkan kursor ke sel ukuran berikutnya secara horizontal.
3. **Kalkulasi Total Otomatis Bebas Alokasi**: Baris total per warna dan total keseluruhan artikel dihitung secara reaktif (`createMemo`) seketika tanpa alokasi memori berulang (*0 unnecessary allocations*).
4. **Penyimpanan Struktur Data**: Data dikirimkan ke backend dalam format array DTO yang bersih dan divalidasi ketat:
   ```json
   {
     "article_name": "Kaos Polo Bordir",
     "color_name": "Hitam Reaktif",
     "size_quantities": [
       { "size_code": "S", "quantity": 12, "surcharge_per_pcs": 0 },
       { "size_code": "M", "quantity": 24, "surcharge_per_pcs": 0 },
       { "size_code": "L", "quantity": 36, "surcharge_per_pcs": 0 },
       { "size_code": "XL", "quantity": 12, "surcharge_per_pcs": 0 },
       { "size_code": "XXL", "quantity": 6, "surcharge_per_pcs": 5000 }
     ]
   }
   ```

---

## 3. Masking & Pemformatan Mata Uang Rupiah (IDR)

- Input nominal rupiah menampilkan format visual yang mudah dibaca pengguna: `Rp 18.500.000`.
- Saat data dikirim ke backend, mask difilter dan dikonversi murni menjadi integer `BIGINT`: `18500000`.
- Dilarang menyimpan nilai desimal atau simbol mata uang ke dalam payload REST.

---

## 4. Validasi Skema Zod Ketat & Tipe Data Terinferensi (Strict Typing)

Contoh validasi pesanan baru dengan aturan pemblokiran ketat:
```typescript
import { z } from 'zod';

export const SizeCodeEnum = z.enum(["S", "M", "L", "XL", "XXL", "3XL", "4XL", "CUSTOM"]);
export type SizeCode = z.infer<typeof SizeCodeEnum>;

export const CreateQuotationSchema = z.object({
  customerId: z.string().uuid("Pilih pelanggan yang valid"),
  articleType: z.enum(["tshirt", "polo", "shirt", "hoodie", "jacket"], {
    errorMap: () => ({ message: "Pilih jenis artikel pakaian yang valid" }),
  }),
  fabricTypeId: z.string().uuid("Pilih jenis bahan baku yang valid"),
  productionDeadline: z.string().refine((val) => new Date(val) > new Date(), {
    message: "Tenggat waktu produksi harus lebih besar dari hari ini",
  }),
  downPaymentAmount: z.number().int().min(100000, "Nominal DP minimal Rp 100.000"),
  items: z.array(z.object({
    color: z.string().trim().min(1, "Warna wajib diisi").max(32),
    sizeBreakdown: z.array(z.object({
      sizeCode: SizeCodeEnum,
      quantity: z.number().int().min(0, "Kuantiti tidak boleh negatif").max(100000),
      surchargePerPcs: z.number().int().min(0, "Surcharge tidak boleh negatif"),
    })).min(1, "Minimal harus ada 1 entri ukuran"),
  })).min(1, "Minimal harus ada 1 varian produk"),
}).strict(); // .strict() menolak key asing di luar skema

export type CreateQuotationInput = z.infer<typeof CreateQuotationSchema>;
```

---

## 5. Referensi & Dokumen Terkait

- 🖥️ Standar Rekayasa Frontend: [frontend.md](./frontend.md)
- ⚙️ Standar Rekayasa Backend: [backend.md](./backend.md)
- 🎨 Standar Visual Desain UI: [ui.md](./ui.md)
- 👆 Panduan UX Lapangan: [ux.md](./ux.md)
- 🧪 Standar QA & Benchmark: [qa.md](./qa.md)
- 📄 Spesifikasi Sales & Order (OMS): [../specs/003a-sales-order-core.md](../specs/003a-sales-order-core.md)
- 📋 Standar Dokumentasi Requirement: [requirements_standard.md](./requirements_standard.md)
