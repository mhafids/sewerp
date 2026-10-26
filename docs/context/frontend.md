# Standar Rekayasa Frontend SolidJS SewERP

Dokumen ini memuat standar arsitektur antarmuka pengguna, reaktivitas, tipe data ketat (*Strict TypeScript*), manajemen state, validasi ketat (*Strict Zod Validation*), serta persyaratan performa **Zero Overhead, Zero Memory Allocation, dan Zero Footprint** untuk SewERP.

---

## Rekomendasi Skills untuk Agent (Recommended Agent Skills)

Ketika agen bertugas membangun komponen antarmuka SolidJS, mengintegrasikan formulir dinamis, atau mengoptimalkan reaktivitas bebas overhead, aktifkan dan kombinasikan skills berikut:
- `ui-ux-pro-max`: Rekayasa komponen Flat UI, konfigurasi utilitas UnoCSS, palet warna Matcha Green & Sakura Pink, dan densitas tinggi tabel garmen.
- `memory-leak-debugging`: Memverifikasi siklus hidup reaktif SolidJS, memastikan pembersihan listener pada `onCleanup()`, serta menganalisis heap snapshot untuk mencegah detached DOM trees.
- `debug-optimize-lcp`: Mengoptimalkan ukuran bundle Vite, code-splitting rute, eliminasi layout shifts (CLS), dan pencapaian waktu muat LCP < 1.2 detik.
- `generative_ui`: Merender widget visual, diagram interaktif alur kerja garmen, dan pratinjau komponen secara inline.
- `a11y-debugging`: Memastikan aksesibilitas keyboard dan standar ARIA pada primitives `@kobalte/core`.

---

## 1. Pilihan Teknologi Frontend

- **Core Library**: [SolidJS](https://www.solidjs.com) v1.9+ (reaktivitas butir-halus / *fine-grained reactivity* tanpa Virtual DOM overhead).
- **Bahasa & Tipe**: TypeScript 5.5+ dengan mode konstelasi **Ultra-Strict Typing**.
- **Build Tool & Runtime**: [Vite](https://vitejs.dev) + [Bun](https://bun.sh) (kecepatan build instan dan eksekusi tes tanpa lag).
- **Styling Engine**: [UnoCSS](https://unocss.dev) dengan preset Mini & Icons (instan utility engine berorientasi Modern Flat UI).
- **Headless UI Primitives**: [@kobalte/core](https://kobalte.dev) (komponen aksesibel ARIA, dialog, dropdown, tabs, popover).
- **Data Fetching & Cache**: [@tanstack/solid-query](https://tanstack.com/query) v5 (manajemen state server, auto-caching, deduplikasi request, dan mutasi optimistik).
- **Routing**: [@solidjs/router](https://github.com/solidjs/solid-router) (SPA client-side routing dengan nested layout).
- **Manajemen Form**: [modular-forms](https://modularforms.dev) + [zod](https://zod.dev) (validasi skema deklaratif type-safe).

---

## 2. Standar TypeScript Super Ketat (Strict Type Requirements)

Konfigurasi `tsconfig.json` frontend SewERP wajib mengaktifkan seluruh flag kepatuhan ketat:

```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictBindCallApply": true,
    "strictPropertyInitialization": true,
    "noImplicitThis": true,
    "alwaysStrict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true
  }
}
```

### Aturan Penulisan Kode TypeScript:
1. **Dilarang Keras Menggunakan Tipe `any`**:
   - Dilarang menuliskan `any` di mana pun dalam kode frontend. Gunakan `unknown` jika tipe belum diketahui, lalu sempitkan dengan *User-Defined Type Guard* (`val is Type`).
2. **Dilarang Type Assertion Serampangan**:
   - Dilarang `as unknown as SomeType` atau `as any`. Seluruh transformasi tipe wajib diverifikasi oleh Zod schema atau guard function runtime.
3. **Discriminated Union untuk State Mesin**:
   - Seluruh state asinkron wajib direpresentasikan dengan union terdiskriminasi:
     ```typescript
     type AsyncState<T> =
       | { status: 'idle' }
       | { status: 'loading' }
       | { status: 'success'; data: T }
       | { status: 'error'; error: ProblemDetails };
     ```

---

## 3. Persyaratan Performa: Zero Overhead, Zero Memory Allocation & Zero Footprint

Workshop konveksi sering menggunakan perangkat tablet murah atau smartphone operasional berkapasitas RAM terbatas (2–3 GB). Frontend SewERP dirancang agar tidak membebani memori browser:

### A. Zero Virtual DOM Overhead (Direct DOM Updates)
- SolidJS tidak menggunakan Virtual DOM. Komponen dieksekusi hanya **satu kali** saat inisialisasi, dan pembaruan UI diarahkan langsung ke teks/atribut node DOM yang presisi melalui kompilasi JSX reaktif.
- Dilarang memecah `props` (*destructuring*) karena akan memutus pelacakan dependensi dan memicu komputasi yang tidak perlu.

### B. Zero Memory Leaks & Zero Footprint (`onCleanup`)
- **Pembersihan Wajib**: Setiap event listener window, timer `setInterval`/`setTimeout`, koneksi WebSocket, atau pemindai kamera barcode wajib dibersihkan di dalam `onCleanup()`:
  ```typescript
  import { onCleanup } from 'solid-js';

  const startScanner = () => {
    const stream = await navigator.mediaDevices.getUserMedia({ video: true });
    // ...
    onCleanup(() => {
      stream.getTracks().forEach(track => track.stop()); // Hentikan kamera & bebaskan memori RAM
    });
  };
  ```
- **Stabilisasi Render List (`<For>`)**: Selalu gunakan komponen `<For>` bawaan SolidJS dengan referensi objek yang stabil untuk merender daftar tabel ukuran atau riwayat pesanan, mencegah *DOM node recreation* dan *memory thrashing*.

### C. Komputasi Bebas Alokasi Ulang (`createMemo`)
- Kalkulasi total harga pesanan, penambahan diskon, dan rekapitulasi kuantiti matriks ukuran wajib dibungkus dalam `createMemo()`. Nilai di-*cache* dan hanya dihitung ulang jika signal asal mengalami perubahan nilai.

---

## 4. Validasi Formulir Ketat (Strict Zod Schemas)

Semua formulir menggunakan validasi Zod dengan aturan pemblokiran ketat:
- **`z.object({...}).strict()`**: Menolak properti tambahan yang tidak terdaftar (*unknown properties*).
- **Validasi Nilai Batas**: Semua angka wajib memiliki batasan eksplisit `.min()` dan `.max()`.
- **Sanitasi Otomatis**: Input string wajib melalui `.trim()`.

Contoh validasi input pesanan baru:
```typescript
export const CreateOrderSchema = z.object({
  customerId: z.string().uuid("ID pelanggan tidak valid"),
  articleType: z.enum(["tshirt", "polo", "shirt", "hoodie", "jacket"]),
  items: z.array(z.object({
    fabricType: z.string().min(2, "Jenis bahan minimal 2 karakter").max(64),
    colorName: z.string().min(2, "Nama warna wajib diisi").max(32),
    baseUnitPrice: z.number().int().positive("Harga satuan harus lebih besar dari 0"),
    sizeBreakdown: z.array(z.object({
      sizeCode: z.enum(["S", "M", "L", "XL", "XXL", "3XL", "4XL", "CUSTOM"]),
      quantity: z.number().int().min(0, "Kuantiti tidak boleh negatif"),
      surchargePerPcs: z.number().int().min(0, "Surcharge tidak boleh negatif"),
    })).min(1, "Minimal harus memilih 1 ukuran"),
  })).min(1, "Minimal harus ada 1 artikel pesanan"),
  downPaymentAmount: z.number().int().positive("Nominal DP harus bernilai positif"),
}).strict();
```

---

## 5. Struktur Direktori Frontend

```text
frontend/src/
├── assets/                  # Ikon SVG, font, gambar statis
├── components/
│   ├── ui/                  # Komponen Flat UI murni (Button, Badge, Card, Modal, Input)
│   └── feedback/            # Skeleton, Toaster, EmptyState, OfflineBanner, ErrorBoundary
├── layouts/
│   ├── app-layout/          # Layout dashboard admin & manajer pabrik (Sidebar + Header)
│   └── operator-layout/     # Layout ringkas fullscreen ramah sentuh untuk area jahit/QC
├── pages/                   # Modul halaman berdasarkan Bounded Context
│   ├── dashboard/           # Widget metrik produksi & omset
│   ├── crm/                 # Master pelanggan & price list
│   ├── oms/                 # Penawaran harga & SPK pesanan
│   ├── production/          # Board tracking 6 stasiun konveksi & scan tiket
│   ├── inventory/           # Stok kain & aksesoris
│   └── payroll/             # Roster penjahit & input borongan harian
├── services/                # HTTP client (fetch/axios wrapper), serializer uang
├── stores/                  # Auth state, Tenant state, Theme store
└── styles/
    ├── tokens.css           # CSS variables (Matcha Green, Sakura Pink, Slate)
    └── main.css             # Base resets & font utilities
```

---

## 6. Referensi & Dokumen Terkait

- 🎨 Standar Visual & Desain UI: [ui.md](./ui.md)
- 👆 Panduan Interaksi & UX Lapangan: [ux.md](./ux.md)
- 📝 Standar Validasi Formulir: [forms.md](./forms.md)
- 🧪 Standar QA & Akses Browser: [qa.md](./qa.md)
- 🏛️ Standar Arsitektur Sistem: [architecture.md](./architecture.md)
