---
name: prezekc-developer
description: >-
  Kerangka kerja (model kerja) membangun aplikasi/sistem informasi dari nol sampai rilis, berbasis tata kelola SI (COBIT 2019: EDM, APO, BAI, DSS, MEA) dan SDLC.
  Aktifkan skill ini ketika pengguna ingin membangun, merancang, mengembangkan, merefaktor, menguji, mengaudit, atau merilis aplikasi/sistem,
  atau mengetik kata kunci seperti "prezekc", "prezekc plan", "prezekc build", "prezekc fix", "prezekc test", "prezekc audit", "prezekc release", "prezekc sync", "lanjut".
---

# Prezekc Developer: Model Kerja Pembangunan Sistem Informasi

Dokumen ini adalah model kerja baku dalam membangun aplikasi/sistem. Setiap proyek dimulai dari **nilai bisnis dan risiko**, bukan dari kode. Pemetaan tata kelola mengacu pada COBIT 2019 (APO, BAI, DSS, MEA) dan dipadukan dengan siklus SDLC.

## Prinsip Inti

1. **Nilai dulu, kode kemudian.** Setiap fitur harus bisa ditelusuri ke tujuan bisnis dan kebutuhan pemangku kepentingan.
2. **Gerbang kualitas (gate).** Tiap fase punya kriteria keluar. Jangan lanjut sebelum gate terpenuhi.
3. **Risiko dikelola sejak awal**, bukan setelah bug muncul.
4. **Zero-regression.** Perbaikan tidak boleh merusak fungsi yang sudah berjalan.
5. **Terdokumentasi dan dapat diaudit.** Keputusan, perubahan, dan hasil uji tercatat.
6. **MVP yang scalable.** Bangun versi minimal yang siap tumbuh, bukan prototipe sekali pakai.
7. **Aman secara bawaan (secure by default).**

## Aturan Perilaku Agen

- Mulai dengan membaca `working.md` (jika ada) dan `docs/` proyek untuk memahami konteks.
- Jika kebutuhan ambigu dan mahal bila salah, tanyakan dulu (maksimal 3 pertanyaan ringkas). Jika jelas, langsung kerjakan.
- Jangan mengarang fitur, data, atau klaim yang tidak diminta. Catat asumsi secara eksplisit.
- Akhiri setiap giliran dengan memperbarui `working.md` (status, keputusan, langkah berikutnya).
- Laporkan hasil singkat: apa yang berubah, bagaimana diuji, risiko tersisa.

---

## Pemetaan Tata Kelola ke Fase Kerja

| Fase | Domain COBIT | Fokus | Keluaran utama |
|---|---|---|---|
| 0. Tata kelola & arah | EDM01, EDM02, EDM03, APO01, APO02 | Nilai, tujuan, strategi, peran | Project Charter, Peta Pemangku Kepentingan |
| 1. Perencanaan & kebutuhan | APO03, APO05, APO06, APO08, APO12 | Arsitektur, portofolio, anggaran, hubungan, risiko | SRS, Register Risiko, Rencana Proyek |
| 2. Desain | BAI02, APO03, APO13 | Kebutuhan terdefinisi, arsitektur, keamanan | Arsitektur, Skema DB, Desain API, Desain UI |
| 3. Pembangunan | BAI03, BAI05, BAI06 | Solusi, perubahan, manajemen perubahan | Kode, Repo, Changelog |
| 4. Pengujian & penerimaan | BAI07, BAI08 | Penerimaan dan transisi perubahan, pengetahuan | Laporan Uji, UAT Sign-off |
| 5. Rilis & operasi | BAI10, DSS01, DSS02, DSS03, DSS04, DSS05 | Konfigurasi, operasi, insiden, masalah, kontinuitas, keamanan | Paket Rilis, Runbook, Rencana Backup |
| 6. Pemantauan & evaluasi | MEA01, MEA02, MEA03, MEA04 | Kinerja, kontrol internal, kepatuhan, jaminan | Laporan Kinerja, Hasil Audit |

> Gunakan hanya proses yang relevan dengan skala proyek. Untuk proyek kecil (UMKM, tugas kuliah, skripsi), cukup versi ringkas pada tiap fase.

---

## Fase 0: Tata Kelola & Arah (EDM, APO01, APO02)

**Tujuan:** memastikan sistem yang dibangun benar-benar memberi nilai dan punya pemilik yang jelas.

**Kerjakan:**
1. Rumuskan masalah bisnis dalam satu kalimat dan tujuan terukur (contoh: "waktu pencatatan pesanan turun dari 10 menit ke 2 menit").
2. Petakan pemangku kepentingan: pemilik, pengguna, pengelola, pihak terdampak.
3. Tetapkan peran dengan RACI (Responsible, Accountable, Consulted, Informed).
4. Tetapkan ruang lingkup (in-scope / out-of-scope) dan kriteria keberhasilan.

**Keluaran:** `docs/00-charter.md` berisi masalah, tujuan, KPI, pemangku kepentingan, RACI, ruang lingkup.

**Gate 0:** tujuan terukur dan pemilik sistem (accountable) teridentifikasi.

---

## Fase 1: Perencanaan & Kebutuhan (APO03, APO05, APO06, APO08, APO12)

**Kerjakan:**
1. **Arsitektur enterprise ringkas (APO03):** proses bisnis yang disentuh, data yang dikelola, sistem lain yang terhubung.
2. **Kebutuhan (APO08, BAI02):** tulis kebutuhan fungsional (FR) dan non-fungsional (NFR: kinerja, keamanan, ketersediaan, usability). Beri ID unik (FR-001, NFR-001) dan prioritas MoSCoW.
3. **Portofolio & anggaran (APO05, APO06):** estimasi biaya, waktu, sumber daya. Pilih fitur MVP.
4. **Manajemen risiko (APO12):** buat register risiko.

   | ID | Risiko | Kemungkinan (1-5) | Dampak (1-5) | Skor | Mitigasi | Pemilik |
   |---|---|---|---|---|---|---|

5. **Rencana proyek (APO11 kualitas, APO07 SDM):** milestone, definisi selesai (Definition of Done), standar kode.

**Keluaran:** `docs/01-srs.md`, `docs/02-risk-register.md`, `docs/03-project-plan.md`.

**Gate 1:** setiap FR bisa ditelusuri ke tujuan di charter; risiko skor tinggi (>=15) punya mitigasi.

---

## Fase 2: Desain (BAI02, APO03, APO13)

**Pikirkan seperti senior systems architect.** Rancang dulu, baru bangun.

**Keluaran wajib:**
- **Arsitektur:** gaya (monolit modular / layanan), lapisan, diagram konteks dan komponen.
- **Struktur file/folder** proyek.
- **Alur data:** DFD level 0 dan 1, sumber dan tujuan data.
- **Skema database:** ERD, tipe data, kunci, indeks, relasi, aturan integritas. Normalisasi minimal 3NF kecuali ada alasan tercatat.
- **Desain API:** endpoint, metode, payload, kode status, format error baku.
- **Arsitektur UI:** peta halaman, alur pengguna, komponen yang dapat dipakai ulang, state (loading, kosong, error).
- **Strategi keamanan (APO13):** autentikasi, otorisasi berbasis peran, validasi input, pengelolaan rahasia, enkripsi data sensitif.
- **Strategi caching dan skalabilitas** bila relevan.

**Keputusan arsitektur** dicatat singkat sebagai ADR (Architecture Decision Record): konteks, keputusan, alternatif, konsekuensi.

**Gate 2:** desain menutup seluruh FR prioritas Must; ada rencana untuk setiap NFR.

---

## Fase 3: Pembangunan (BAI03, BAI05, BAI06)

**Standar kerja:**
- Bangun **MVP tipis end-to-end** dulu (satu alur lengkap dari UI sampai database), baru lebar.
- Kode modular, konsisten, mudah dibaca. Hindari duplikasi.
- Semua akses database memakai prepared statement/parameterized query.
- Penanganan error terlindungi dengan `try-catch`; jangan membocorkan detail teknis ke pengguna.
- Tidak ada kredensial hardcoded. Gunakan variabel lingkungan atau file konfigurasi di luar repo (`.env` masuk `.gitignore`).
- Validasi input di sisi server, bukan hanya di klien.
- Komit kecil dan bermakna. Format pesan: `tipe(ruang): ringkasan` (contoh: `feat(order): tambah validasi stok`).
- Catat perubahan di `CHANGELOG.md` (manajemen perubahan, BAI06).

**Komponen UI (produksi):** dapat dipakai ulang, aksesibel (label, kontras, navigasi keyboard), responsif, dengan state loading/kosong/error/sukses.

**Gate 3:** fitur MVP berjalan, tidak ada kredensial di kode, sintaks/lint lolos.

---

## Fase 4: Pengujian & Penerimaan (BAI07, BAI08)

**Lapisan uji:**
1. **Statis:** lint, cek sintaks, cek variabel belum diinisialisasi.
2. **Unit/fungsional:** per fitur, termasuk **edge case** (input kosong, ekstrem, duplikat, karakter khusus, akses tanpa izin).
3. **Integrasi:** alur lintas modul dan database.
4. **Keamanan:** SQL injection, XSS, CSRF, IDOR, validasi unggah file (cek tipe MIME sebenarnya, batasi ukuran, ubah nama file).
5. **Kinerja:** waktu respons, query lambat (N+1), penggunaan memori.
6. **UAT:** diuji pengguna nyata terhadap skenario di SRS. Catat hasil dan tanda terima.

**Matriks keterlacakan:** FR-ID -> kasus uji -> hasil (Lulus/Gagal).

**Gate 4:** semua FR Must lulus, tidak ada cacat kritis/tinggi terbuka, UAT disetujui pemilik.

---

## Fase 5: Rilis & Operasi (BAI10, DSS01-DSS05)

**Kerjakan:**
1. **Paket rilis bersih:** tanpa berkas pengembangan, rahasia, atau data uji. Hitung checksum (SHA256) dan catat di `RELEASE_MANIFEST.md` bersama versi (SemVer).
2. **Konfigurasi (BAI10):** daftar komponen, versi, dependensi, lingkungan.
3. **Operasi (DSS01):** runbook (cara deploy, restart, cek kesehatan).
4. **Insiden & masalah (DSS02, DSS03):** alur pelaporan, tingkat keparahan, analisis akar masalah.
5. **Kontinuitas (DSS04):** jadwal backup, uji pemulihan, rencana rollback.
6. **Keamanan operasional (DSS05):** hak akses minimum, patch, log akses, pembatasan folder sensitif (mis. `.htaccess` di `uploads/`).

**Gate 5:** rollback teruji, backup terjadwal, runbook tersedia.

---

## Fase 6: Pemantauan & Evaluasi (MEA)

**Kerjakan:**
- **MEA01:** ukur KPI dari charter (contoh: waktu proses, jumlah error, ketersediaan).
- **MEA02:** tinjau kontrol internal (akses, perubahan, backup) berkala.
- **MEA03:** cek kepatuhan terhadap regulasi/kebijakan yang berlaku (mis. UU PDP bila mengelola data pribadi).
- **MEA04:** audit/penjaminan independen untuk sistem berisiko tinggi.
- Masukkan hasil evaluasi ke backlog perbaikan.

**Keluaran:** `docs/06-evaluation.md` (KPI aktual vs target, temuan, tindak lanjut).

---

## Pintasan Perintah (Shortcuts)

Jika pengguna mengetik kata kunci berikut, agen WAJIB langsung menjalankan alurnya secara mandiri.

### 1. `prezekc plan [nama sistem]`
- **Artinya:** susun fondasi tata kelola dan perencanaan (Fase 0-1).
- **Tindakan:**
  1. Buat `docs/00-charter.md`, `docs/01-srs.md`, `docs/02-risk-register.md`.
  2. Ajukan maksimal 3 pertanyaan klarifikasi bila tujuan atau ruang lingkup belum jelas.
  3. Laporkan ringkasan tujuan, MVP, dan 3 risiko teratas.

### 2. `prezekc design`
- **Artinya:** rancang arsitektur lengkap (Fase 2).
- **Tindakan:** hasilkan arsitektur, struktur file, ERD/skema DB, desain API, arsitektur UI, strategi keamanan, dan ADR.

### 3. `prezekc build [fitur]`
- **Artinya:** bangun fitur/MVP sesuai desain (Fase 3).
- **Tindakan:** implementasikan irisan end-to-end, uji sintaks, perbarui `CHANGELOG.md` dan `working.md`.

### 4. `prezekc fix [isu/pesan error]`
- **Artinya:** perbaiki bug sebagai senior debugging engineer.
- **Tindakan:**
  1. Reproduksi dan temukan lokasi penyebab.
  2. Jelaskan: fungsi kode, apa masalahnya, kenapa gagal, edge case terkait.
  3. Perbaiki dengan prinsip zero-regression dan uji ulang sampai hilang.
  4. Laporkan akar masalah dan perbaikan secara ringkas.

### 5. `prezekc test`
- **Artinya:** inspeksi dan pengujian menyeluruh (Fase 4).
- **Tindakan:** audit sintaks file yang berubah, cek penanganan error, jalankan uji fungsional/edge case, perbarui matriks keterlacakan, nyatakan siap rilis atau tidak.

### 6. `prezekc audit`
- **Artinya:** cek kepatuhan keamanan dan tata kelola.
- **Tindakan:** periksa checklist di bawah (Checklist Kepatuhan) dan berikan hasil Lulus/Gagal per butir beserta rekomendasi.

### 7. `prezekc refactor`
- **Artinya:** pahami codebase lalu tingkatkan kualitas tanpa mengubah fungsi.
- **Tindakan:** ringkas arsitektur dan alur data; identifikasi masalah struktural, kode duplikat, bottleneck, risiko maintainability; usulkan strategi; terapkan perbaikan bertahap dengan uji tiap langkah.

### 8. `prezekc optimize`
- **Artinya:** optimasi kecepatan, memori, skalabilitas.
- **Tindakan:** temukan bottleneck dan logika tidak efisien; usulkan strategi; terapkan dan ukur sebelum/sesudah.

### 9. `prezekc release`
- **Artinya:** siapkan rilis (Fase 5).
- **Tindakan:** bangun paket bersih, hitung SHA256, catat versi di `RELEASE_MANIFEST.md`, siapkan catatan rilis dan rencana rollback.

### 10. `prezekc sync` / `lanjut`
- **Artinya:** sinkronkan status dan lanjutkan pekerjaan terakhir.
- **Tindakan:** baca `working.md` (`<in_progress>`), lanjutkan langkah berikutnya tanpa bertanya ulang, uji, lalu perbarui `<completed_tasks>` dan `<in_progress>`.

---

## Mode Peran (Prompt Siap Pakai)

Gunakan sesuai kebutuhan.

**Aplikasi lengkap dari nol**
> Berpikirlah seperti senior full-stack engineer yang membangun aplikasi lengkap siap produksi. Rancang arsitektur dulu, lalu bangun versi minimal tapi scalable. Hasil: arsitektur, struktur file, skema database, endpoint API, arsitektur UI, kode lengkap.

**Memahami dan merefaktor codebase**
> Berpikirlah seperti senior engineer yang baru bergabung ke codebase asing. Pahami arsitektur dan alur data, identifikasi masalah struktural, kode duplikat, bottleneck, risiko maintainability. Hasil: ringkasan arsitektur, area bermasalah, strategi refaktor, kode perbaikan. Fungsionalitas tetap sama.

**Senior debugging engineer**
> Analisis kode dengan teliti, berpikir langkah demi langkah, temukan akar masalah. Hasil: fungsi kode, masalah, kenapa gagal, edge case, kode perbaikan siap produksi.

**Multi-agent**
> Berperan sebagai 4 agen: Architect (merancang), Engineer (membangun), Reviewer (kontrol kualitas), Optimizer (kinerja). Hasil: arsitektur, implementasi, masukan review, versi final teroptimasi.

---

## Checklist Kepatuhan (untuk `prezekc audit`)

**Tata kelola**
- [ ] Charter berisi tujuan terukur, pemilik, ruang lingkup
- [ ] Setiap FR terlacak ke tujuan dan ke kasus uji
- [ ] Register risiko ada dan diperbarui
- [ ] Keputusan arsitektur utama tercatat (ADR)

**Keamanan (APO13, DSS05)**
- [ ] Tidak ada kredensial/rahasia hardcoded; `.env` di `.gitignore`
- [ ] Query memakai prepared statement; error database dibungkus `try-catch`
- [ ] Validasi input di sisi server; output di-escape (cegah XSS)
- [ ] Proteksi CSRF pada aksi yang mengubah data
- [ ] Otorisasi berbasis peran diterapkan di server (bukan hanya menyembunyikan menu)
- [ ] Unggah file divalidasi tipe MIME sebenarnya, ukuran dibatasi, folder `uploads/` tidak dapat mengeksekusi skrip
- [ ] Kata sandi disimpan dengan hash modern (bcrypt/argon2)
- [ ] Folder sensitif dibatasi aksesnya

**Kualitas & perubahan (BAI)**
- [ ] Sintaks/lint lolos, tidak ada variabel belum diinisialisasi
- [ ] `CHANGELOG.md` diperbarui; versi mengikuti SemVer
- [ ] Uji fungsional dan edge case terdokumentasi
- [ ] UAT disetujui pemilik

**Operasi & kontinuitas (DSS)**
- [ ] Runbook dan rencana rollback tersedia
- [ ] Backup terjadwal dan pernah diuji pemulihannya
- [ ] Log akses/kesalahan aktif tanpa mencatat data sensitif
- [ ] Paket rilis bersih dan tercatat SHA256-nya

**Pemantauan (MEA)**
- [ ] KPI diukur terhadap target di charter
- [ ] Temuan evaluasi masuk ke backlog

---

## Format `working.md` (Memori Proyek)

```markdown
<project>
Nama sistem, tujuan, stack, versi saat ini
</project>

<decisions>
- [tanggal] Keputusan penting dan alasannya
</decisions>

<completed_tasks>
- [x] Tugas selesai (referensi FR-ID)
</completed_tasks>

<in_progress>
- [ ] Tugas berjalan, langkah berikutnya, kendala
</in_progress>

<risks>
- Risiko terbuka dan mitigasinya
</risks>
```

## Struktur Repo yang Disarankan

```
proyek/
├── docs/
│   ├── 00-charter.md
│   ├── 01-srs.md
│   ├── 02-risk-register.md
│   ├── 03-project-plan.md
│   ├── 04-architecture.md      (termasuk ADR)
│   ├── 05-test-report.md
│   └── 06-evaluation.md
├── src/
├── tests/
├── .env.example
├── CHANGELOG.md
├── RELEASE_MANIFEST.md
└── working.md
```

## Larangan

- Jangan membangun fitur di luar ruang lingkup tanpa persetujuan pemilik.
- Jangan melewati gate; jika terpaksa, catat sebagai risiko dengan pemilik dan tenggat.
- Jangan menyimpan kredensial atau data pribadi nyata di repo.
- Jangan mengubah fungsi yang sudah berjalan tanpa uji regresi.
- Jangan mengklaim "siap produksi" tanpa hasil uji dan audit yang tercatat.
