# Prezekc Developer

Kerangka kerja (model kerja) dalam bentuk **skill** untuk membangun aplikasi/sistem informasi dari nol sampai rilis. Dirancang agar agen AI (Claude dan sejenisnya) bekerja seperti tim pengembang yang terkelola: mulai dari nilai bisnis dan risiko, bukan langsung dari kode.

Kerangka ini memadukan:

- **Tata kelola SI (COBIT 2019):** EDM, APO, BAI, DSS, MEA
- **SDLC:** perencanaan, desain, pembangunan, pengujian, rilis, operasi
- **Pintasan perintah semi-otonom** (`prezekc run`, `fix`, `test`, `pack`, dst.)
- **Standar kualitas UI** agar antarmuka punya karakter produk, bukan tampilan generik buatan AI

Seluruh isi kerangka ada di [`SKILL.md`](./SKILL.md).

---

## Untuk siapa

- Mahasiswa dan pengembang yang membangun sistem informasi (skripsi, tugas, UMKM, proyek organisasi) dan ingin proses yang rapi serta terdokumentasi.
- Siapa pun yang bekerja dengan agen AI dan ingin hasil yang konsisten, aman, dan dapat diaudit.

## Prinsip inti

1. Nilai dulu, kode kemudian
2. Gerbang kualitas (gate) di setiap fase
3. Risiko dikelola sejak awal
4. Zero-regression
5. Terdokumentasi dan dapat diaudit
6. MVP yang scalable
7. Aman secara bawaan
8. UI dirancang, bukan dihasilkan asal

## Alur kerja

| Fase | Domain COBIT | Keluaran utama |
|---|---|---|
| 0. Tata kelola & arah | EDM01-03, APO01-02 | Project Charter, peta pemangku kepentingan |
| 1. Perencanaan & kebutuhan | APO03, 05, 06, 08, 12 | SRS, register risiko, rencana proyek |
| 2. Desain | BAI02, APO03, APO13 | Arsitektur, skema DB, desain API, desain UI |
| 3. Pembangunan | BAI03, 05, 06 | Kode, repo, changelog |
| 4. Pengujian & penerimaan | BAI07-08 | Laporan uji, UAT |
| 5. Rilis & operasi | BAI10, DSS01-05 | Paket rilis, runbook, rencana backup |
| 6. Pemantauan & evaluasi | MEA01-04 | Laporan kinerja, hasil audit |

Setiap fase punya **gate**: jangan lanjut sebelum kriteria keluarnya terpenuhi. Untuk proyek kecil, pakai versi ringkas tiap fase.

## Pintasan perintah

Ketik kata kunci berikut ke agen, dan agen langsung menjalankan alurnya secara mandiri.

| Perintah | Fungsi |
|---|---|
| `prezekc plan [nama sistem]` | Susun charter, SRS, dan register risiko |
| `prezekc design` | Rancang arsitektur, skema DB, API, dan UI |
| `prezekc build [fitur]` | Bangun MVP/fitur end-to-end |
| `prezekc run` / `lanjut` | Lanjutkan pekerjaan terakhir dari `working.md` |
| `prezekc fix [isu]` | Perbaiki bug sampai akar masalah beres |
| `prezekc test` | Inspeksi dan pengujian menyeluruh |
| `prezekc audit` | Audit kepatuhan keamanan dan tata kelola |
| `prezekc pack` | Buat paket rilis bersih, SHA256, dan `RELEASE_MANIFEST.md` |
| `prezekc sync` | Sinkronkan status proyek di `working.md` |
| `prezekc refactor` | Pahami codebase dan tingkatkan kualitas tanpa ubah fungsi |
| `prezekc optimize` | Optimasi kecepatan, memori, skalabilitas |
| `prezekc ui [layar]` | Rapikan/bangun UI dengan standar kualitas UI |

Selain pintasan, tersedia tujuh **mode peran** (full-stack, refaktor, debugging, arsitek sistem, performa, multi-agent, komponen UI) di bagian 5 `SKILL.md`.

## Cara memakai

> Lokasi folder skill dapat berbeda antarversi. Cek dokumentasi resmi aplikasi yang kamu pakai bila langkah di bawah tidak cocok.

**Claude Code**
1. Salin repo ini ke folder skill, contoh untuk satu proyek:
   ```bash
   mkdir -p .claude/skills/prezekc-developer
   cp SKILL.md .claude/skills/prezekc-developer/
   ```
   Untuk semua proyek, gunakan `~/.claude/skills/prezekc-developer/`.
2. Mulai sesi dan ketik, misalnya, `prezekc plan sistem pemesanan fotokopi`.

**Claude.ai**
1. Kemas folder skill (berisi `SKILL.md`) menjadi ZIP.
2. Unggah lewat pengaturan Skills, lalu aktifkan.

**Agen/editor lain**
Tempel isi `SKILL.md` sebagai instruksi sistem atau berkas aturan proyek.

## Struktur repo

```
prezekc-developer/
├── SKILL.md      # isi kerangka kerja (dibaca agen)
└── README.md     # dokumen ini
```

Saat dipakai pada proyek nyata, kerangka ini menyarankan struktur berikut di proyekmu:

```
proyek/
├── docs/            # charter, SRS, risiko, arsitektur, uji, evaluasi
├── src/
├── tests/
├── CHANGELOG.md
├── RELEASE_MANIFEST.md
└── working.md       # memori proyek: tugas selesai, berjalan, risiko
```

## Menyesuaikan

- **Nama pintasan:** ganti awalan `prezekc` di `SKILL.md` (bagian `description` dan bagian 2).
- **Stack:** perintah uji bersifat netral; sesuaikan dengan stack proyek (PHP, Node, Python, dsb.).
- **Skala:** hapus proses COBIT yang tidak relevan untuk proyek kecil.

## Kredit

Standar kualitas UI (bagian 4 `SKILL.md`) diadaptasi dari skill **unslop** karya Damir Sokolovsky ([@heydamir](https://x.com/heydamir)), berlisensi [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Kredit ini wajib dipertahankan pada salinan atau adaptasi.

Kerangka tata kelola mengacu pada COBIT 2019 (ISACA). Repo ini bukan produk resmi ISACA.

## Lisensi

Tentukan lisensi repo ini (misalnya MIT) dengan menambahkan berkas `LICENSE`. Bagian UI yang diadaptasi dari unslop tetap tunduk pada CC BY 4.0.
