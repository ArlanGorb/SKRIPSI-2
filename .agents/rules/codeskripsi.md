---
trigger: always_on
---

# Project Rules — SIM Persuratan Distan Minahasa Selatan

> Tempelkan isi file ini ke **Antigravity → Rules → Project Rules** (bukan Global Rules), supaya berlaku otomatis di setiap sesi agent tanpa perlu diulang. File lengkap spesifikasi ada di `docs/master-prompt.md` di root proyek — rujuk ke sana untuk detail yang tidak tercakup di sini.

## Tech Stack (wajib)
- Next.js (App Router) + TypeScript
- Supabase (PostgreSQL, Auth dengan JWT, Storage, REST/GraphQL otomatis)
- Tailwind CSS + shadcn/ui
- Zod + React Hook Form untuk validasi
- Deploy: Vercel (frontend + API Routes) + Supabase (backend/DB)
- Frontend TIDAK PERNAH mengakses database langsung — selalu lewat Next.js API Routes atau Supabase client dengan RLS aktif

## Bahasa & Domain
- SELURUH teks UI, label, pesan error, notifikasi, dan laporan WAJIB Bahasa Indonesia. Jangan buat i18n multi-bahasa.
- Ini sistem internal instansi pemerintah (Dinas Pertanian), bukan aplikasi publik/konsumen.

## 6 Peran (RBAC — wajib diterapkan di UI DAN RLS database, bukan hanya UI)
1. **Administrator** — kelola akun & data master saja. Tidak melihat isi surat, tidak terlibat disposisi/tanda tangan.
2. **Petugas Surat/Agenda** — catat surat masuk/keluar, upload PDF, kelola arsip. Tidak berwenang disposisi/paraf/ttd.
3. **Sekretaris Dinas** — paraf koordinasi surat keluar, pantau status.
4. **Kepala Dinas** — beri disposisi awal, tanda tangani surat keluar, lihat semua rekap.
5. **Kepala Bidang** (5 bidang: Prasarana & Sarana, Tanaman Pangan & Hortikultura, Peternakan, Perkebunan, Penyuluhan) — terima & teruskan disposisi ke Staf Pelaksana bidangnya SAJA.
6. **Staf Pelaksana Bidang** — tindak lanjuti disposisi bidangnya SAJA.

Kepala Bidang & Staf Pelaksana wajib dibatasi RLS ke bidangnya sendiri — jangan pernah tampilkan/izinkan akses lintas bidang tanpa disposisi eksplisit.

## Aturan Bisnis yang Tidak Boleh Disederhanakan
- **Nomor surat keluar** format `{kode_klasifikasi}/{nomor_urut}/DISTAN-MS/{bulan_romawi}/{tahun}`. WAJIB dijalankan dalam transaksi DB dengan row locking (`SELECT ... FOR UPDATE`) + unique constraint pada kolom nomor, agar tidak ada nomor ganda saat concurrent request. Counter reset ke 1 tiap tahun baru.
- **Disposisi berjenjang**: Kepala Dinas → (Sekretaris Dinas dan/atau Kepala Bidang) → Staf Pelaksana. Simpan riwayat lengkap (waktu, pelaku, catatan) di setiap perpindahan status — ini audit trail, jangan di-skip.
- **Paraf/tanda tangan** = autentikasi akun + catatan riwayat approval di sistem. BUKAN sertifikat digital signature.
- **Notifikasi email** hanya untuk 5 event: surat masuk baru perlu disposisi, disposisi diteruskan, permintaan paraf/ttd, reminder tenggat, overdue+eskalasi ke atasan.

## Batasan — JANGAN bangun ini kecuali diminta eksplisit
- Integrasi sistem persuratan instansi lain/nasional
- Digital signature tersertifikasi
- OCR / klasifikasi surat berbasis AI
- Aplikasi mobile native (web-responsive saja)
- Modul kepegawaian/keuangan/administrasi lain
- Bahasa selain Indonesia

## Cara Kerja Agent
- Jalankan di **Planning Mode** dulu untuk setiap fase besar — tampilkan rencana/artifact sebelum eksekusi, saya akan review sebelum menyetujui.
- Untuk keputusan yang berdampak besar dan tidak diatur di sini (struktur folder detail, penyedia email pihak ketiga, strategi CI/CD) — **tanya dulu**, jangan berasumsi diam-diam.
- Setelah tiap modul selesai, jalankan self-test via browser (kalau memungkinkan) dan berikan ringkasan cara saya bisa memverifikasi manual.
- Gunakan Supabase MCP server (bukan raw SQL manual di luar migrasi) untuk perubahan schema, agar migrasi tetap tercatat dan konsisten.
