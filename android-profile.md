# Profil Android: Flutter + SQLite (Offline-First) + MySQL

Profil stack untuk `prezekc-developer` saat membangun aplikasi Android (APK/AAB). Berkas ini melengkapi `SKILL.md`: fase, gate, dan standar UI di sana tetap berlaku; di sini hanya ditentukan **cara menerapkannya pada stack ini**.

Cara memakai: simpan di samping `SKILL.md` (mis. `references/android-profile.md`) dan tambahkan baris ini di `SKILL.md`:

> Untuk proyek aplikasi Android (Flutter), ikuti juga `references/android-profile.md`.

---

## 1. Stack

| Komponen | Peran |
|---|---|
| **Android Studio** | IDE utama: editor, emulator (Device Manager), debugging, profiler. Pasang plugin Flutter dan Dart |
| **Flutter (Dart)** | SDK dan bahasa untuk antarmuka dan logika aplikasi |
| **SQLite** | Basis data lokal di perangkat; sumber data utama saat offline (arsitektur **Offline-First**) |
| **MySQL** | Basis data server pusat; data terintegrasi dan tujuan sinkronisasi |
| **API server** | Lapisan penghubung antara aplikasi dan MySQL (wajib; lihat bagian 3) |

> Aplikasi **tidak boleh terhubung langsung ke MySQL**. Kredensial basis data tidak boleh ada di APK karena APK dapat dibongkar. Semua akses lewat API ber-autentikasi.

Pilihan teknologi API server (PHP, Node, dsb.) dan paket Flutter (state management, SQLite helper) diputuskan di Fase 2 dan dicatat sebagai ADR. Pilihan yang umum dipakai:
- SQLite: `sqflite` atau `drift` (type-safe, migrasi lebih rapi)
- HTTP: `dio` atau `http`
- Status koneksi: `connectivity_plus`
- Penyimpanan rahasia (token): `flutter_secure_storage`
- State management: Riverpod, Bloc, atau Provider (pilih satu, konsisten)

---

## 2. Arsitektur Offline-First

Prinsip: **UI selalu membaca dan menulis ke SQLite lokal.** Jaringan hanya urusan sinkronisasi di latar belakang. Aplikasi harus tetap berfungsi penuh tanpa internet.

```
UI (Widget)
   │
State management (Riverpod/Bloc/Provider)
   │
Repository ──────────────┐
   │                     │
Local data source     Sync service ── Remote data source (API)
(SQLite)              (outbox, pull/push)        │
                                              MySQL (server)
```

Struktur folder (feature-first):

```
lib/
├── core/            # tema, konstanta, util, error, jaringan, db helper
├── data/
│   ├── local/       # SQLite: tabel, DAO, migrasi
│   ├── remote/      # klien API, DTO
│   └── repositories/
├── sync/            # outbox, sync service, resolusi konflik
├── features/
│   └── <fitur>/     # presentation (UI), state, domain
└── main.dart
test/                # unit + widget test
integration_test/    # uji alur end-to-end
```

---

## 3. Rancangan Basis Data dan Sinkronisasi

### Aturan skema (SQLite dan MySQL dibuat selaras)

- **Primary key berupa UUID** (dibuat di perangkat), bukan auto-increment, agar data yang dibuat offline tidak bentrok antar perangkat.
- Setiap tabel yang disinkronkan punya kolom: `id` (UUID), `created_at`, `updated_at`, `deleted_at` (soft delete, NULL jika aktif), `version` (opsional, untuk deteksi konflik).
- Tabel lokal tambahan: `sync_status` per baris (`synced` / `pending` / `failed`) **atau** tabel **outbox**.
- **Hapus = soft delete** (isi `deleted_at`), agar penghapusan ikut tersinkron.
- MySQL: engine InnoDB, charset `utf8mb4`, kunci asing dan indeks (`updated_at`, kolom pencarian), normalisasi minimal 3NF.
- SQLite: aktifkan foreign key (`PRAGMA foreign_keys = ON`), **versi skema + migrasi berurutan** (`onUpgrade`), jangan mengubah skema tanpa migrasi.

### Alur sinkronisasi

1. **Tulis lokal dulu.** Setiap perubahan disimpan ke SQLite, lalu dicatat di outbox (operasi: create/update/delete, id, payload, waktu, jumlah percobaan).
2. **Push.** Saat online, kirim outbox berurutan ke API. Sertakan **idempotency key** (id operasi) agar pengiriman ulang tidak menggandakan data.
3. **Pull inkremental.** Minta perubahan server sejak `last_sync_at` (cursor `since`), terapkan ke SQLite dalam satu transaksi.
4. **Resolusi konflik.** Default: *last-write-wins* berdasarkan `updated_at` **waktu server** (jangan percaya jam perangkat). Untuk data kritis (stok, saldo, status pesanan), gunakan `version` dan tolak penulisan usang, lalu minta pengguna memilih atau gabung otomatis menurut aturan bisnis. Tetapkan aturannya per tabel di dokumen desain.
5. **Retry dengan backoff** saat gagal; tandai `failed` setelah batas percobaan dan tampilkan status ke pengguna.
6. Picu sinkronisasi: saat aplikasi dibuka, saat koneksi kembali, setelah perubahan, dan berkala (mis. WorkManager lewat paket `workmanager`) bila diperlukan.

### Keamanan sinkronisasi dan API

- HTTPS saja; autentikasi token (mis. JWT/token sesi) dengan masa berlaku dan refresh.
- Otorisasi di **server** per peran dan per kepemilikan data (cegah IDOR). Jangan percaya data dari klien.
- Validasi input di server; query MySQL memakai prepared statement.
- Token disimpan di `flutter_secure_storage`, bukan SharedPreferences biasa.
- Data sangat sensitif di SQLite lokal: pertimbangkan enkripsi (mis. SQLCipher).
- Tidak ada kunci API atau kredensial hardcoded di kode Dart; konfigurasi per lingkungan lewat `--dart-define` atau berkas yang tidak masuk repo.

---

## 4. Pemetaan Pintasan `prezekc` ke Flutter

| Pintasan | Tindakan pada stack ini |
|---|---|
| `prezekc plan` | Tambahkan ke SRS: kebutuhan offline (fitur mana yang harus jalan tanpa internet), kebijakan konflik, perangkat target dan versi Android minimum |
| `prezekc design` | Hasilkan: ERD SQLite dan MySQL (selaras), kontrak API (endpoint sinkronisasi push/pull), diagram alur sinkronisasi, struktur folder Flutter, peta layar, ADR |
| `prezekc build` | Irisan end-to-end: layar -> state -> repository -> SQLite -> outbox -> API -> MySQL. Bangun satu fitur penuh dulu, lalu lebarkan |
| `prezekc run` / `lanjut` | Baca `working.md`, lanjutkan, lalu `flutter analyze` dan `flutter test` |
| `prezekc fix` | Reproduksi di emulator/perangkat, baca log (`flutter logs`/Logcat), perbaiki, uji ulang. Untuk bug sinkronisasi, uji skenario offline -> online |
| `prezekc test` | Lihat bagian 5 |
| `prezekc audit` | Lihat checklist bagian 7 |
| `prezekc pack` | Lihat bagian 6 |
| `prezekc sync` | Perbarui `working.md` (jangan tertukar dengan fitur sinkronisasi data aplikasi) |
| `prezekc refactor` / `optimize` | Pisahkan lapisan (UI, state, repository, data), hilangkan rebuild widget berlebih, kurangi query lambat, gunakan `const`, `ListView.builder`, indeks SQLite |
| `prezekc ui` | Terapkan standar UI (SKILL.md bagian 4) pada Flutter, lihat bagian 8 |

---

## 5. Pengujian (Fase 4)

Perintah umum:

```bash
flutter analyze                     # analisis statis
dart format --set-exit-if-changed . # cek format
flutter test                        # unit + widget test
flutter test integration_test       # uji end-to-end (butuh emulator/perangkat)
```

Cakupan uji khusus stack ini:

1. **Unit:** repository, DAO SQLite (pakai basis data uji in-memory), logika resolusi konflik, pembuat payload outbox.
2. **Sinkronisasi (kritis):**
   - Buat/ubah/hapus data saat **offline**, lalu online: data sampai ke MySQL tepat satu kali (idempotensi).
   - Dua perangkat mengubah data yang sama: konflik diselesaikan sesuai aturan.
   - Koneksi putus di tengah sinkronisasi: tidak ada data hilang atau ganda; retry berjalan.
   - Pull setelah lama tidak sinkron: data besar tetap konsisten.
3. **Migrasi SQLite:** uji upgrade dari tiap versi skema lama ke terbaru tanpa kehilangan data.
4. **Widget:** state loading, kosong, error, offline, dan "sedang sinkron".
5. **Keamanan:** token tidak tersimpan di tempat tidak aman, akses data milik pengguna lain ditolak server, APK rilis tidak memuat rahasia.
6. **Perangkat nyata:** uji minimal pada satu perangkat fisik berspesifikasi rendah dan satu versi Android minimum yang didukung.

Matriks keterlacakan tetap: FR-ID -> kasus uji -> hasil.

---

## 6. Rilis APK/AAB (`prezekc pack`, Fase 5)

1. **Versi:** naikkan `version: x.y.z+build` di `pubspec.yaml` (SemVer; angka setelah `+` adalah build number yang harus selalu naik).
2. **Penandatanganan:** buat keystore sekali. Simpan `key.properties` dan berkas keystore **di luar repo** (masuk `.gitignore`); cadangkan keystore di tempat aman (kehilangan keystore = tidak bisa memperbarui aplikasi di Play Store dengan kunci yang sama).
3. **Build:**

   ```bash
   flutter clean && flutter pub get
   flutter build apk --release                  # satu APK
   flutter build apk --release --split-per-abi  # APK per arsitektur (lebih kecil)
   flutter build appbundle --release            # AAB untuk Google Play
   ```

   Keluaran biasanya di `build/app/outputs/flutter-apk/` (APK) dan `build/app/outputs/bundle/release/` (AAB).
4. **Perkecil dan lindungi:** aktifkan R8/shrink dan obfuscation bila sesuai (`--obfuscate --split-debug-info=<folder>`), simpan simbol debug untuk membaca crash.
5. **Integritas:** hitung SHA256 berkas rilis dan catat bersama versi di `RELEASE_MANIFEST.md`:

   ```bash
   sha256sum build/app/outputs/flutter-apk/app-release.apk
   # Windows PowerShell: Get-FileHash <berkas> -Algorithm SHA256
   ```
6. **Konfigurasi lingkungan:** pastikan URL API menunjuk ke server produksi (HTTPS) dan mode debug mati.
7. **Server:** skema MySQL produksi sudah termigrasi, backup MySQL terjadwal, API punya rate limit dan log.
8. **Uji pasca-build:** pasang APK rilis (bukan debug) di perangkat fisik, jalankan alur utama termasuk skenario offline-online.
9. **Rollback:** simpan APK/AAB versi stabil sebelumnya; API harus tetap kompatibel dengan versi aplikasi lama (versioning API, jangan hapus kolom/endpoint mendadak).

---

## 7. Checklist Audit Khusus Android/Flutter

**Arsitektur dan data**
- [ ] UI hanya membaca/menulis ke SQLite; tidak ada panggilan jaringan langsung dari widget
- [ ] Primary key UUID; kolom `updated_at` dan `deleted_at` ada di tabel yang disinkronkan
- [ ] Outbox atau `sync_status` ada; operasi sinkronisasi idempoten
- [ ] Aturan konflik per tabel terdokumentasi dan teruji
- [ ] Migrasi SQLite berurutan dan teruji
- [ ] Skema SQLite dan MySQL selaras (nama kolom, tipe, aturan)

**Keamanan**
- [ ] Aplikasi tidak terhubung langsung ke MySQL; tidak ada kredensial DB di APK
- [ ] Semua lalu lintas HTTPS; token di `flutter_secure_storage`
- [ ] Otorisasi dan validasi dilakukan di server; query memakai prepared statement
- [ ] Tidak ada kunci API/rahasia hardcoded; keystore dan `key.properties` tidak masuk repo
- [ ] Izin Android (`AndroidManifest.xml`) hanya yang diperlukan
- [ ] Log rilis tidak memuat data sensitif
- [ ] Build rilis tidak dalam mode debug

**Kualitas**
- [ ] `flutter analyze` tanpa peringatan penting; format konsisten
- [ ] Uji unit, widget, dan sinkronisasi lulus
- [ ] Teruji di perangkat fisik dan versi Android minimum
- [ ] State loading/kosong/error/offline/sinkron tersedia di UI

**Rilis**
- [ ] `pubspec.yaml` versi dan build number dinaikkan
- [ ] SHA256 dan versi tercatat di `RELEASE_MANIFEST.md`
- [ ] Keystore tercadangkan; APK/AAB versi sebelumnya disimpan untuk rollback
- [ ] Backup MySQL terjadwal dan pernah diuji pemulihannya

---

## 8. Standar UI pada Flutter

Terapkan standar UI `SKILL.md` bagian 4 dengan padanan berikut:

- **Spasi:** kelipatan 4 (`const SizedBox(height: 8)`, `EdgeInsets.all(16)`); definisikan konstanta jarak di `core/`.
- **Warna dan tipografi:** satu `ThemeData` (Material 3) dengan `ColorScheme` dan `TextTheme`; jangan menulis warna/ukuran huruf langsung di widget. Kontras teks minimal 4,5:1.
- **Ikon:** satu set ikon dan satu ukuran (mis. 20-24 dp); tanpa emoji sebagai ikon.
- **Target sentuh:** minimal 48x48 dp; fokus dan label semantik untuk aksesibilitas (`Semantics`), dukung skala teks sistem.
- **Komponen:** widget reusable di `core/widgets`, dengan state loading, kosong, error, dan offline; tampilkan **indikator status sinkronisasi** yang jelas dan tenang (ini cocok dijadikan "tanda khas" aplikasi).
- **Layout:** responsif untuk berbagai lebar layar (`LayoutBuilder`/`MediaQuery`), uji layar kecil dan orientasi lanskap.
- **Gerak:** durasi pendek dan satu kurva; hindari animasi berulang yang tidak perlu.
- **Copy:** bahasa Indonesia jelas, tanpa tanda seru berlebihan; pesan error menjelaskan langkah selanjutnya (mis. "Tidak ada koneksi. Data disimpan dan akan dikirim saat online").

---

## 9. Alat dan Persiapan Lingkungan

- Pasang Flutter SDK dan Android Studio; di Android Studio pasang plugin **Flutter** dan **Dart**.
- Jalankan `flutter doctor` dan selesaikan semua temuan (Android SDK, lisensi: `flutter doctor --android-licenses`).
- Buat emulator di **Device Manager**; uji juga di perangkat fisik (aktifkan USB debugging).
- Gunakan **DevTools** (Flutter) dan **Profiler** (Android Studio) untuk analisis performa dan memori.
- Kunci versi Flutter dan dependensi (`pubspec.lock` masuk repo) agar build dapat diulang.
- Tiap proyek menyimpan dokumen keputusan stack di `docs/04-architecture.md` (ADR): paket yang dipilih dan alasannya.

## 10. Catatan

- Sinkronisasi dua arah dengan banyak perangkat adalah bagian tersulit dari arsitektur ini. Mulai dari kasus paling sederhana (satu pengguna, satu perangkat, data milik pengguna itu), uji kuat, baru lebarkan.
- Jika data tidak pernah diubah oleh lebih dari satu perangkat, resolusi konflik cukup sederhana (server menang atau last-write-wins).
- Saran paket dan perintah di atas adalah titik awal. Periksa dokumentasi resmi Flutter, Android, dan paket terkait untuk versi terbaru sebelum dipakai.
