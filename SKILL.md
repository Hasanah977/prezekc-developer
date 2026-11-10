---
name: prezekc-developer
description: >-
  Kerangka kerja (model kerja) membangun aplikasi/sistem informasi dari nol sampai rilis, berbasis tata kelola SI (COBIT 2019: EDM, APO, BAI, DSS, MEA) dan SDLC, lengkap dengan pintasan perintah semi-otonom dan standar kualitas UI.
  Aktifkan skill ini ketika pengguna ingin membangun, merancang, mengembangkan, merefaktor, men-debug, mengoptimasi, menguji, mengaudit, merilis, atau merapikan tampilan aplikasi/sistem,
  atau mengetik kata kunci singkat seperti "prezekc", "prezekc plan", "prezekc design", "prezekc build", "prezekc run", "prezekc fix", "prezekc test", "prezekc audit", "prezekc pack", "prezekc sync", "prezekc refactor", "prezekc optimize", "prezekc ui", "lanjut", atau "lanjutkan".
---

# Prezekc Developer: Model Kerja Pembangunan Sistem Informasi

Dokumen ini adalah model kerja baku dalam membangun aplikasi/sistem. Setiap proyek dimulai dari **nilai bisnis dan risiko**, bukan dari kode. Tata kelola mengacu pada COBIT 2019 (EDM, APO, BAI, DSS, MEA), dipadukan dengan SDLC, pintasan perintah semi-otonom, dan standar kualitas UI.

Isi dokumen:
1. Prinsip inti dan aturan perilaku agen
2. Pintasan perintah (shortcuts)
3. Pemetaan tata kelola ke fase kerja (Fase 0-6)
4. Standar kualitas UI (anti-"slop")
5. Mode peran (prompt siap pakai)
6. Checklist kepatuhan, format `working.md`, struktur repo, larangan

---

## 1. Prinsip Inti

1. **Nilai dulu, kode kemudian.** Setiap fitur harus bisa ditelusuri ke tujuan bisnis dan kebutuhan pemangku kepentingan.
2. **Gerbang kualitas (gate).** Tiap fase punya kriteria keluar. Jangan lanjut sebelum gate terpenuhi.
3. **Risiko dikelola sejak awal**, bukan setelah bug muncul.
4. **Zero-regression.** Perbaikan tidak boleh merusak fungsi yang sudah berjalan.
5. **Terdokumentasi dan dapat diaudit.** Keputusan, perubahan, dan hasil uji tercatat.
6. **MVP yang scalable.** Bangun versi minimal yang siap tumbuh, bukan prototipe sekali pakai.
7. **Aman secara bawaan (secure by default).**
8. **UI dirancang, bukan dihasilkan asal.** Tampilan mengikuti karakter produk (lihat bagian 4).

## Aturan Perilaku Agen

- Mulai dengan membaca `working.md` (jika ada) dan `docs/` untuk memahami konteks.
- Jika kebutuhan ambigu dan mahal bila salah, tanyakan dulu (maksimal 3 pertanyaan ringkas). Jika jelas, langsung kerjakan.
- Jangan mengarang fitur, data, atau klaim yang tidak diminta. Catat asumsi secara eksplisit.
- Akhiri setiap giliran dengan memperbarui `working.md` (status, keputusan, langkah berikutnya).
- Laporkan hasil singkat: apa yang berubah, bagaimana diuji, risiko tersisa.
- Sesuaikan perintah uji dengan stack proyek (contoh PHP: `php -l`; Node: `npm test`/lint; Python: `pytest`/`ruff`).

---

## 2. Pintasan Perintah (Shortcuts Mapping)

Jika pengguna mengetik kata kunci berikut, agen WAJIB langsung menjalankan alurnya secara mandiri.

### 1. `prezekc run` atau `lanjut` / `lanjutkan`
- **Artinya:** lanjutkan pekerjaan terakhir yang belum selesai.
- **Tindakan Otonom AI:**
  1. Baca `working.md` pada blok `<in_progress>`.
  2. Lanjutkan langkah berikutnya tanpa bertanya ulang.
  3. Uji sintaks dan verifikasi fungsionalitas.
  4. Perbarui status `working.md` di akhir giliran.

### 2. `prezekc plan [nama sistem]`
- **Artinya:** susun fondasi tata kelola dan perencanaan (Fase 0-1).
- **Tindakan:**
  1. Buat `docs/00-charter.md`, `docs/01-srs.md`, `docs/02-risk-register.md`.
  2. Ajukan maksimal 3 pertanyaan klarifikasi bila tujuan atau ruang lingkup belum jelas.
  3. Laporkan ringkasan tujuan, MVP, dan 3 risiko teratas.

### 3. `prezekc design`
- **Artinya:** rancang sistem sebagai senior systems architect (Fase 2).
- **Tindakan:** hasilkan arsitektur, struktur komponen, alur data, desain API, skema database, strategi caching, strategi keamanan, dan ADR.

### 4. `prezekc build [fitur]`
- **Artinya:** bangun fitur/MVP sesuai desain (Fase 3).
- **Tindakan:** implementasikan irisan end-to-end, uji sintaks, perbarui `CHANGELOG.md` dan `working.md`.

### 5. `prezekc fix [isu/pesan error]`
- **Artinya:** perbaiki bug atau error tertentu secara mandiri (*Loop-Resolution*).
- **Tindakan Otonom AI:**
  1. Cari lokasi file penyebab error (grep/view file) dan reproduksi bila bisa.
  2. Jelaskan: fungsi kode, apa masalahnya, kenapa gagal, edge case terkait.
  3. Perbaiki dengan prinsip *Zero-Regression*.
  4. Uji ulang sampai error hilang.
  5. Laporkan akar masalah dan perbaikan secara ringkas.

### 6. `prezekc test`
- **Artinya:** inspeksi dan pengujian menyeluruh (Fase 4).
- **Tindakan Otonom AI:**
  1. Audit sintaks/lint pada seluruh file yang baru diubah.
  2. Verifikasi tidak ada variabel yang belum diinisialisasi.
  3. Pastikan penanganan error (mis. PDO/DB) terlindungi `try-catch`.
  4. Jalankan uji fungsional dan edge case; perbarui matriks keterlacakan FR -> uji -> hasil.
  5. Laporkan: siap produksi atau ada yang perlu diperbaiki.

### 7. `prezekc pack`
- **Artinya:** buat paket rilis bersih untuk deployment server (Fase 5).
- **Tindakan Otonom AI:**
  1. Jalankan skrip build rilis proyek (mis. `build_upload.ps1` atau `build_release.sh`); jika belum ada, buat.
  2. Ambil hash SHA256 dari paket ZIP yang dihasilkan.
  3. Catat versi rilis baru (SemVer) dan SHA256 ke `RELEASE_MANIFEST.md`.
  4. Siapkan catatan rilis dan rencana rollback.
  5. Laporkan nama file zip rilis dan hash-nya.

### 8. `prezekc sync`
- **Artinya:** sinkronkan status memori proyek saat ini.
- **Tindakan Otonom AI:**
  1. Periksa file apa saja yang baru dimodifikasi.
  2. Sinkronkan dan perbarui blok `<completed_tasks>` dan `<in_progress>` di `working.md`.
  3. Tampilkan ringkasan singkat status proyek.

### 9. `prezekc audit`
- **Artinya:** pemeriksaan kepatuhan keamanan, tata kelola, dan standar arsitektur.
- **Tindakan Otonom AI:**
  1. Periksa ketiadaan kredensial hardcoded.
  2. Periksa `.htaccess`/aturan server pada folder `uploads/` dan folder sensitif.
  3. Periksa validasi unggah file (mis. PHP: `finfo_file()` untuk tipe MIME sebenarnya).
  4. Berikan checklist kepatuhan (bagian 6) dengan hasil Lulus/Gagal per butir dan rekomendasi.

### 10. `prezekc refactor`
- **Artinya:** pahami codebase asing lalu tingkatkan kualitas tanpa mengubah fungsi.
- **Tindakan:** ringkas arsitektur dan alur data; identifikasi masalah struktural, kode duplikat, bottleneck, risiko maintainability; usulkan strategi; terapkan bertahap dengan uji tiap langkah.

### 11. `prezekc optimize`
- **Artinya:** optimasi kecepatan, memori, skalabilitas.
- **Tindakan:** temukan bottleneck, logika tidak efisien, render tidak perlu; usulkan strategi; terapkan dan ukur sebelum/sesudah.

### 12. `prezekc ui [layar/komponen]`
- **Artinya:** rapikan atau bangun antarmuka agar punya karakter produk, bukan tampilan generik buatan AI.
- **Tindakan:** jalankan alur "Standar Kualitas UI" (bagian 4): audit tanda slop, tulis Direction lima baris, lalu perbaiki dalam sembilan lapisan berurutan.

---

## 3. Pemetaan Tata Kelola ke Fase Kerja

| Fase | Domain COBIT | Fokus | Keluaran utama |
|---|---|---|---|
| 0. Tata kelola & arah | EDM01, EDM02, EDM03, APO01, APO02 | Nilai, tujuan, strategi, peran | Project Charter, Peta Pemangku Kepentingan |
| 1. Perencanaan & kebutuhan | APO03, APO05, APO06, APO08, APO12 | Arsitektur, portofolio, anggaran, hubungan, risiko | SRS, Register Risiko, Rencana Proyek |
| 2. Desain | BAI02, APO03, APO13 | Kebutuhan terdefinisi, arsitektur, keamanan | Arsitektur, Skema DB, Desain API, Desain UI |
| 3. Pembangunan | BAI03, BAI05, BAI06 | Solusi, perubahan, manajemen perubahan | Kode, Repo, Changelog |
| 4. Pengujian & penerimaan | BAI07, BAI08 | Penerimaan dan transisi perubahan, pengetahuan | Laporan Uji, UAT Sign-off |
| 5. Rilis & operasi | BAI10, DSS01-DSS05 | Konfigurasi, operasi, insiden, masalah, kontinuitas, keamanan | Paket Rilis, Runbook, Rencana Backup |
| 6. Pemantauan & evaluasi | MEA01-MEA04 | Kinerja, kontrol internal, kepatuhan, jaminan | Laporan Kinerja, Hasil Audit |

> Gunakan hanya proses yang relevan dengan skala proyek. Untuk proyek kecil (UMKM, tugas kuliah, skripsi), cukup versi ringkas tiap fase.

### Fase 0: Tata Kelola & Arah (EDM, APO01, APO02)
**Tujuan:** memastikan sistem memberi nilai dan punya pemilik yang jelas.
1. Rumuskan masalah bisnis dalam satu kalimat dan tujuan terukur (contoh: "waktu pencatatan pesanan turun dari 10 menit ke 2 menit").
2. Petakan pemangku kepentingan: pemilik, pengguna, pengelola, pihak terdampak.
3. Tetapkan peran dengan RACI (Responsible, Accountable, Consulted, Informed).
4. Tetapkan ruang lingkup (in/out-of-scope) dan kriteria keberhasilan.

**Keluaran:** `docs/00-charter.md`. **Gate 0:** tujuan terukur dan pemilik sistem (accountable) teridentifikasi.

### Fase 1: Perencanaan & Kebutuhan (APO03, APO05, APO06, APO08, APO12)
1. **Arsitektur enterprise ringkas (APO03):** proses bisnis yang disentuh, data yang dikelola, sistem lain yang terhubung.
2. **Kebutuhan (APO08, BAI02):** kebutuhan fungsional (FR) dan non-fungsional (NFR: kinerja, keamanan, ketersediaan, usability). Beri ID unik (FR-001, NFR-001) dan prioritas MoSCoW.
3. **Portofolio & anggaran (APO05, APO06):** estimasi biaya, waktu, sumber daya; tetapkan fitur MVP.
4. **Manajemen risiko (APO12):** register risiko.

   | ID | Risiko | Kemungkinan (1-5) | Dampak (1-5) | Skor | Mitigasi | Pemilik |
   |---|---|---|---|---|---|---|

5. **Rencana proyek (APO11 kualitas, APO07 SDM):** milestone, Definition of Done, standar kode.

**Keluaran:** `docs/01-srs.md`, `docs/02-risk-register.md`, `docs/03-project-plan.md`.
**Gate 1:** setiap FR terlacak ke tujuan di charter; risiko skor >=15 punya mitigasi.

### Fase 2: Desain (BAI02, APO03, APO13)
**Pikirkan seperti senior systems architect.** Rancang dulu, baru bangun.
- **Arsitektur:** gaya (monolit modular / layanan), lapisan, diagram konteks dan komponen.
- **Struktur file/folder** proyek.
- **Alur data:** DFD level 0 dan 1.
- **Skema database:** ERD, tipe data, kunci, indeks, relasi, integritas. Minimal 3NF kecuali ada alasan tercatat.
- **Desain API:** endpoint, metode, payload, kode status, format error baku.
- **Arsitektur UI:** peta halaman, alur pengguna, komponen reusable, state (loading, kosong, error). Tulis **Direction UI** (bagian 4).
- **Strategi keamanan (APO13):** autentikasi, otorisasi berbasis peran, validasi input, pengelolaan rahasia, enkripsi data sensitif.
- **Strategi caching dan skalabilitas** bila relevan.
- Catat keputusan penting sebagai **ADR** (konteks, keputusan, alternatif, konsekuensi).

**Gate 2:** desain menutup seluruh FR Must; ada rencana untuk setiap NFR.

### Fase 3: Pembangunan (BAI03, BAI05, BAI06)
- Bangun **MVP tipis end-to-end** dulu (satu alur lengkap dari UI sampai database), baru lebar.
- Kode modular, konsisten, mudah dibaca, tanpa duplikasi.
- Akses database memakai prepared statement/parameterized query.
- Penanganan error terlindungi `try-catch`; jangan bocorkan detail teknis ke pengguna.
- Tidak ada kredensial hardcoded; gunakan variabel lingkungan/file konfigurasi di luar repo (`.env` masuk `.gitignore`).
- Validasi input di server, bukan hanya di klien.
- Komit kecil dan bermakna: `tipe(ruang): ringkasan` (contoh `feat(order): tambah validasi stok`).
- Catat perubahan di `CHANGELOG.md` (BAI06).
- Komponen UI produksi: reusable, aksesibel, responsif, dengan state loading/kosong/error/sukses, dan mengikuti bagian 4.

**Gate 3:** MVP berjalan, tidak ada kredensial di kode, sintaks/lint lolos.

### Fase 4: Pengujian & Penerimaan (BAI07, BAI08)
1. **Statis:** lint, sintaks, variabel belum diinisialisasi.
2. **Unit/fungsional:** per fitur dan **edge case** (input kosong, ekstrem, duplikat, karakter khusus, akses tanpa izin).
3. **Integrasi:** alur lintas modul dan database.
4. **Keamanan:** SQL injection, XSS, CSRF, IDOR, validasi unggah file.
5. **Kinerja:** waktu respons, query lambat (N+1), memori.
6. **UI:** responsif, keyboard/fokus, kontras, state kosong/error.
7. **UAT:** uji pengguna nyata terhadap skenario SRS; catat hasil dan tanda terima.

**Matriks keterlacakan:** FR-ID -> kasus uji -> hasil.
**Gate 4:** semua FR Must lulus, tidak ada cacat kritis/tinggi terbuka, UAT disetujui pemilik.

### Fase 5: Rilis & Operasi (BAI10, DSS01-DSS05)
1. **Paket rilis bersih** (`prezekc pack`): tanpa berkas pengembangan, rahasia, atau data uji; SHA256 dan versi di `RELEASE_MANIFEST.md`.
2. **Konfigurasi (BAI10):** daftar komponen, versi, dependensi, lingkungan.
3. **Operasi (DSS01):** runbook (deploy, restart, cek kesehatan).
4. **Insiden & masalah (DSS02, DSS03):** alur pelaporan, tingkat keparahan, analisis akar masalah.
5. **Kontinuitas (DSS04):** jadwal backup, uji pemulihan, rencana rollback.
6. **Keamanan operasional (DSS05):** hak akses minimum, patch, log akses, pembatasan folder sensitif.

**Gate 5:** rollback teruji, backup terjadwal, runbook tersedia.

### Fase 6: Pemantauan & Evaluasi (MEA)
- **MEA01:** ukur KPI dari charter (waktu proses, jumlah error, ketersediaan).
- **MEA02:** tinjau kontrol internal (akses, perubahan, backup) berkala.
- **MEA03:** cek kepatuhan regulasi/kebijakan (mis. UU PDP bila mengelola data pribadi).
- **MEA04:** audit/penjaminan independen untuk sistem berisiko tinggi.
- Masukkan temuan ke backlog perbaikan. **Keluaran:** `docs/06-evaluation.md`.

---

## 4. Standar Kualitas UI (Anti-"Slop")

> Bagian ini diadaptasi dari skill **unslop** karya Damir Sokolovsky ([@heydamir](https://x.com/heydamir)), lisensi CC BY 4.0. Kredit ini wajib dipertahankan pada salinan atau adaptasi.

UI hasil generasi AI cenderung gagal dengan cara yang sama: semua permukaan berpendar, tiap label berisi emoji, jarak asal-asalan, teks berteriak. Tujuannya: **fitur, konten, dan urutan bagian tetap; ukuran, warna, kata, dan gambar dibuat disengaja.** Kekayaan visual ditaruh pada satu-dua "momen" konten yang diingat orang, sedangkan kerangka antarmuka (chrome) dibuat tenang.

### Alur (`prezekc ui`)
1. **Lihat layar.** Jalankan aplikasi dan ambil tangkapan layar bila bisa; nilai tampilan nyata, bukan hanya kode.
2. **Audit.** Daftar setiap tanda slop (gradien ungu, glow, teks gradien, emoji sebagai ikon, lencana "New ✨", blob mengilap, label huruf besar berjarak lebar, elemen berdenyut, jarak tidak seragam), satu baris per temuan beserta lokasinya.
3. **Tentukan besarnya perubahan.**
   - *Slop berat:* jalankan seluruh proses.
   - *Slop ringan:* mode poles; perbaiki hanya temuan, pertahankan palet, tipografi, dan tata letak.
   - *Tidak ada slop:* katakan apa adanya, usulkan maksimal 2-3 penyempurnaan kecil. Jangan ubah layar yang sudah baik hanya agar berbeda.
4. **Tetapkan Direction lalu berhenti** untuk konfirmasi pengguna sebelum mengedit (kecuali pengguna sudah meminta langsung kerjakan).
5. **Perbaiki dalam sembilan lapisan berurutan** (selesaikan satu lapisan di seluruh layar sebelum lapisan berikutnya).
6. **Cek** dengan checklist UI (bagian 6), render ulang, bandingkan dengan daftar audit.
7. **Laporkan** direction dan ringkasan tiap lapisan.

### Direction (tulis lima baris, turunkan dari produknya, bukan dari tren)
- **Produk:** apa yang dilakukan dan untuk siapa, satu kalimat.
- **Karakter:** dua-tiga kata konkret (mis. "tenang, presisi, sedikit hangat"). Dilarang: bersih, modern, sleek, minimalis, premium.
- **Dunia:** tempat/material alami produk (aplikasi catatan dekat kertas dan tinta; alat monitoring dekat instrumen dan shift malam). Menentukan nuansa netral, aksen, dan visual.
- **Momen:** satu-dua tempat yang boleh kaya warna dan detail (grafik utama, objek nyata dalam pratinjau).
- **Tanda khas:** satu detail kecil khas produk, dipakai konsisten (titik status hidup, keycap, garis rute).

Jika produk sudah punya brand asli (logo, warna, font) yang bukan slop, pertahankan dan bangun direction di sekitarnya.

### Sembilan lapisan
1. **Spasi.** Semua margin, padding, gap, tinggi tetap kelipatan 4px. Benda yang sama berukuran sama. Jarak dalam grup lebih kecil dari jarak antar grup. Ikon, label, angka sejajar satu kolom. Padding: panel/sidebar ~16, kartu 20-24, tepi halaman 24-32. Baris navigasi tinggi 32-36 dengan jarak 2-4. Hapus nilai sekali pakai seperti `mt-[13px]`.
2. **Warna.** Satu ramp netral bernuansa dunia produk; tiga peran teks (utama, sekunder, redup); satu aksen yang jarang dipakai (aksi utama, pilihan, angka terpenting). Pada aplikasi/dasbor, latar dan permukaan tetap netral; warna kaya hanya di momen. Kontras teks di atas aksen minimal 4,5:1. Hapus gradien antar-hue, teks gradien, glow berwarna, bayangan berwarna, blob, border neon. Border berupa garis tipis transparansi 6-10%. Warna semantik (hijau baik, merah error) diredam.
3. **Ikon.** Emoji sebagai ikon diganti ikon garis dari satu set (pakai set proyek, atau lucide). Satu ukuran (16/18px) dan satu tebal garis (1,5). Warna ikon mengikuti peran teksnya. Lebar kotak ikon sama di tiap baris. Buang ikon yang tidak menambah apa-apa.
4. **Teks/copy.** Judul menyatakan apa yang dilakukan produk untuk siapa, dengan kata biasa. Tanpa tanda seru, tanpa judul sapaan ("Selamat datang kembali! 👋"; ganti dengan nama konteks seperti "November 2026"). Label pendek ("Cari", bukan "Cari apa saja..."). Perubahan menyebut pembandingnya ("+12% vs bulan lalu"). Judul, label, tombol, keterangan pendek tanpa titik akhir. Tanpa kata hype (supercharge, seamless, magic, revolusioner). Klaim yang tidak bisa diverifikasi ("10x lebih cepat", "dipercaya 10.000+ tim") dibuang kecuali pengguna memastikan benar. Data contoh harus tampak nyata, bukan `[X]` atau `123,456`. Lencana "PRO/New/Beta" dibuang kecuali memang perlu.
5. **Tipografi.** Pilih huruf sesuai karakter; satu keluarga untuk antarmuka plus paling banyak satu pendamping (mono untuk angka/meta atau display untuk judul besar). Beberapa ukuran dari satu skala, bobot 400 dan 500 (600 untuk penekanan langka); tanpa bobot 700-900 pada judul. Hierarki dari ukuran dan warna, bukan bobot. Sentence case (bukan `NAVIGASI` berjarak lebar). Tanpa letter-spacing negatif pada teks kecil. Angka memakai `tabular-nums` dan satu format di seluruh layar. Tanpa italic untuk penekanan.
6. **Tata letak.** Pertahankan bagian dan urutannya. Aplikasi/dasbor berproporsi aplikasi nyata: sidebar 240-280, konten fluid (maks ~1440-1600), jangan diperkecil jadi kolom sempit di tengah. Hero landing: adegan/gambar tunggal dengan ruang kosong tenang untuk judul, atau split asimetris (visual minimal setengah lebar); hindari semuanya bertumpuk dan rata tengah. Judul panjang rata kiri. Satu aksi utama dan paling banyak satu aksi sekunder. Navigasi tenang: logo, tautan yang ada, satu tombol. Layar pertama tersusun utuh tanpa pita kosong. Di layar kecil semuanya bertumpuk sesuai urutan baca, teks tidak di bawah 11px.
7. **Detail.** Radius bersarang: radius dalam = radius luar - padding (minimal 4); kontrol 8-10, kartu 12-16, panel 14-20; tidak semua berbentuk pil. Hitungan/meta menjadi teks redup, bukan pil berpendar. Pintasan keyboard menjadi keycap kecil. Grafik: grid redup, label mono redup, satu grafik utama paling besar (yang lain berbeda jenis, mis. daftar berurut atau tabel pendek), satu aksen per grafik, tipe sesuai data (tren = garis, perbandingan = batang, porsi = batang bertumpuk/daftar berbatang, target = progress bar). Kartu statistik hanyalah angka: label, angka besar, perubahan beserta pembanding; tanpa sparkline/ikon hiasan. Variasikan ukuran kartu. Status aktif = langkah permukaan kecil dengan highlight tipis, bukan gradien berglow. Ada state hover dan fokus yang terlihat. Logo = nama produk dalam huruf utama (plus tanda sederhana), bukan wordmark gradien atau ubin berkilau. Tanda khas muncul di tempat yang tepat dan tidak di tempat lain.
8. **Visual.** Buang hiasan pengisi (blob mengilap, orb, kubus melayang, grid berpendar, otak holografik). Hero landing butuh visual yang menunjukkan hal nyata, mis. pratinjau produk berbasis kode dengan data masuk akal. Dasbor/aplikasi hampir tidak perlu gambar buatan; momennya datang dari produk (grafik yang digarap rapi, angka tertata, konten asli, status hidup). Gambar buatan hanya untuk empty state, onboarding, momen pemasaran, atau jika diminta. Tanpa alat pembuat gambar: bangun yang bisa dengan kode, tulis prompt untuk sisanya, dan sisakan slot berukuran tepat.
9. **Gerak.** Hapus gerak berulang tanpa alasan (`animate-pulse`/`animate-ping` pada hal yang tidak live, shimmer, glow bernapas, elemen melayang, parallax), `transition-all`, spring memantul, `hover:scale` pada kartu. Pertahankan gerak yang menjelaskan sesuatu: hover berupa perubahan warna/permukaan cepat, tekan = skala turun sedikit, menu muncul dengan fade dan naik beberapa piksel. Transisikan hanya warna, latar, border, opacity, transform (sebut eksplisit); jangan animasikan width, height, top, left, margin, atau blur besar. Satu kurva easing dan beberapa durasi. Paling banyak satu animasi masuk saat load.

### Larangan UI
- Jangan mengganti nama produk atau mengubah apa produk itu. Logo asli dipertahankan; logo slop digambar ulang datar.
- Jangan mengganti dekorasi yang dibuang dengan dekorasi baru, dan jangan menukar satu tren dengan tren lain (glass, grain, noise, neon, bento).
- Jangan mengarang fitur, klaim, atau data.
- Jangan jatuh ke tampilan yang sama untuk semua produk. Direction yang menentukan palet, huruf, dan visual.
- **Uji template:** jika nama produk ditukar dengan produk lain dan layar masih cocok, dorong direction lebih kuat pada copy, visual, dan tanda khas.

---

## 5. Mode Peran (Prompt Siap Pakai)

Gunakan sesuai kebutuhan, juga bisa dipicu lewat pintasan di bagian 2.

**1. Aplikasi lengkap dari nol**
> Berpikirlah seperti senior full-stack engineer yang membangun aplikasi lengkap siap produksi. Pertama, rancang arsitektur sistem, lalu bangun versi minimal tapi scalable. Hasilnya harus mencakup: arsitektur, struktur file, skema database, endpoint API, arsitektur UI, kode lengkap. Rancang seperti MVP startup sungguhan dan buat scalable.

**2. Memahami dan merefaktor codebase**
> Berpikirlah seperti senior engineer yang baru bergabung ke codebase besar yang asing. Pertama, pahami arsitektur dan alur datanya. Lalu identifikasi masalah struktural, kode duplikat, bottleneck performa, risiko maintainability. Hasil: ringkasan arsitektur, area bermasalah, strategi refaktor, kode yang diperbaiki. Fungsionalitas tetap sama, kualitas ditingkatkan.

**3. Senior debugging engineer**
> Berpikirlah seperti senior debugging engineer yang menyelidiki bug di lingkungan produksi. Analisis kode dengan teliti, berpikir langkah demi langkah, temukan akar masalah, usulkan solusi yang kokoh. Hasil: fungsi kode, apa masalahnya, kenapa gagal, edge case, kode perbaikan siap produksi.

**4. Desain sistem + implementasi**
> Berpikirlah seperti senior systems architect. Rancang sistem scalable untuk produk ini, lalu bangun versi produksi minimalnya. Mencakup: arsitektur, struktur komponen, alur data, desain API, skema database, strategi caching, kode implementasi.

**5. Optimasi performa**
> Berpikirlah seperti performance engineer yang mengoptimalkan kode. Tujuan: kecepatan, penggunaan memori, skalabilitas. Cari bottleneck, logika tidak efisien, rendering tidak perlu. Hasil: masalah performa, strategi optimasi, kode yang diperbaiki.

**6. Alur kerja multi-agent**
> Kamu adalah 4 agen yang berkolaborasi: Architect (merancang sistem), Engineer (membangun), Reviewer (kontrol kualitas), Optimizer (meningkatkan performa). Hasil: arsitektur, implementasi, masukan review, versi final yang dioptimalkan.

**7. Komponen UI level produksi**
> Berpikirlah seperti senior frontend engineer dan bangun komponen UI yang reusable, aksesibel, siap produksi. Pertimbangkan loading state, edge case, desain responsif, aksesibilitas. Hasil: arsitektur komponen, desain props, implementasi, contoh penggunaan. Ikuti standar kualitas UI (bagian 4).

---

## 6. Checklist Kepatuhan (untuk `prezekc audit`)

**Tata kelola**
- [ ] Charter berisi tujuan terukur, pemilik, ruang lingkup
- [ ] Setiap FR terlacak ke tujuan dan ke kasus uji
- [ ] Register risiko ada dan diperbarui
- [ ] Keputusan arsitektur utama tercatat (ADR)

**Keamanan (APO13, DSS05)**
- [ ] Tidak ada kredensial/rahasia hardcoded; `.env` di `.gitignore`
- [ ] Query memakai prepared statement; error database dibungkus `try-catch`
- [ ] Validasi input di server; output di-escape (cegah XSS)
- [ ] Proteksi CSRF pada aksi yang mengubah data
- [ ] Otorisasi berbasis peran diterapkan di server, bukan hanya menyembunyikan menu
- [ ] Unggah file divalidasi tipe MIME sebenarnya, ukuran dibatasi, nama diacak; `uploads/` tidak dapat mengeksekusi skrip (`.htaccess` atau setara)
- [ ] Kata sandi di-hash dengan algoritma modern (bcrypt/argon2)
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
- [ ] Paket rilis bersih dan SHA256-nya tercatat di `RELEASE_MANIFEST.md`

**Pemantauan (MEA)**
- [ ] KPI diukur terhadap target di charter
- [ ] Temuan evaluasi masuk ke backlog

**UI (bagian 4)**
- [ ] Direction tertulis dan semua lapisan mengikutinya
- [ ] Semua nilai spasi di skala 4px; benda sama berukuran sama
- [ ] Ikon, label, dan angka sejajar dalam kolom
- [ ] Satu ramp netral, tiga peran teks, satu aksen berkontras cukup
- [ ] Tanpa gradien antar-hue, teks gradien, glow, bayangan berwarna, blob
- [ ] Tanpa emoji di antarmuka; satu set ikon, satu ukuran dan tebal garis
- [ ] Huruf sesuai karakter, bobot 400-600, tanpa label huruf besar berjarak, tanpa italic, tanpa tracking negatif pada teks kecil
- [ ] Tanpa tanda seru, tanpa judul sapaan, tanpa titik akhir pada teks pendek, tanpa klaim tak terverifikasi
- [ ] Angka satu format dengan tabular figures
- [ ] Radius bersarang konsisten; tombol berbagi radius kontrol
- [ ] Hero/layar utama punya visual nyata atau pratinjau produk, bukan pengisi
- [ ] Tanpa gerak berulang atau memantul; satu kurva easing; tanpa animasi properti layout
- [ ] Uji template lolos (layar tidak cocok untuk produk lain)

---

## 7. Format `working.md` (Memori Proyek)

```markdown
<project>
Nama sistem, tujuan, stack, versi saat ini
</project>

<direction_ui>
Produk / Karakter / Dunia / Momen / Tanda khas
</direction_ui>

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

## 8. Struktur Repo yang Disarankan

```
proyek/
├── docs/
│   ├── 00-charter.md
│   ├── 01-srs.md
│   ├── 02-risk-register.md
│   ├── 03-project-plan.md
│   ├── 04-architecture.md      (termasuk ADR dan Direction UI)
│   ├── 05-test-report.md
│   └── 06-evaluation.md
├── src/
├── tests/
├── .env.example
├── CHANGELOG.md
├── RELEASE_MANIFEST.md
└── working.md
```

## 9. Larangan

- Jangan membangun fitur di luar ruang lingkup tanpa persetujuan pemilik.
- Jangan melewati gate; jika terpaksa, catat sebagai risiko dengan pemilik dan tenggat.
- Jangan menyimpan kredensial atau data pribadi nyata di repo.
- Jangan mengubah fungsi yang sudah berjalan tanpa uji regresi.
- Jangan mengklaim "siap produksi" tanpa hasil uji dan audit yang tercatat.
- Jangan menghapus kredit pembuat unslop pada bagian 4.
