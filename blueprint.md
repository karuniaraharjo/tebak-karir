# Blueprint — Tebak Karir Informatika (Scan Wajah Gimik)
### Dokumen Master untuk Membangun Website Interaktif Pameran Prodi Informatika dengan AI Coding Agent

> **Status dokumen:** living document. Versi 1.0 — 8 November 2026.
> **Cara pakai singkat:** upload file ini utuh ke AI coding agent di awal sesi, lalu katakan: *"Jadikan dokumen ini sumber kebenaran utama proyek. Jika ada bagian yang tidak jelas, tanyakan ke saya sebelum menulis kode. Kerjakan bertahap sesuai Roadmap (Bagian 28)."*
>
> **Penting dan jujur sejak awal:** website ini adalah **hiburan/gimik** untuk pameran. Hasil karir **tidak** dihitung dari analisis wajah yang sebenarnya, dan wajah tidak bisa memprediksi karir seseorang. Kamera hanya dipakai agar pengalaman terasa interaktif. Pengunjung harus diberi tahu hal ini di layar (lihat FR-003).

---

## Peta Struktur

```text
BAGIAN A — Fondasi Proyek           (1–4)
BAGIAN B — Requirements             (5–8)
BAGIAN C — Desain                   (9–10)
BAGIAN D — Arsitektur Teknis        (11–16)
BAGIAN E — Kualitas & Keamanan      (17–21)
BAGIAN F — Operasional              (22–25)  ← diringkas, proyek kecil
BAGIAN G — Cara AI Bekerja          (26–30)
LAMPIRAN A — Daftar 50 Karir + Deskripsi (data konten utama)
LAMPIRAN B — Bank Teks (scan, hasil, error, disclaimer)
LAMPIRAN C — Prompt Siap Pakai untuk AI Agent
```

---

# BAGIAN A — Fondasi Proyek

## 1. Project Overview

- **Nama (working title):** *Tebak Karir Informatika* — judul tampilan boleh diganti panitia (**TBD:** nama resmi acara/stand).
- **Masalah yang dipecahkan:** stand prodi di pameran butuh atraksi yang membuat pengunjung (calon mahasiswa, pelajar, tamu) berhenti, ikut berinteraksi, dan mengenal ragam karir di bidang Informatika.
- **Solusi:** website kiosk satu layar. Pengunjung melakukan "scan wajah" (gimik), sistem menampilkan satu karir Informatika yang dipilih acak lengkap dengan deskripsi lucu bergaya "kamu cocok jadi … karena …", statistik palsu, level kelangkaan, dan kartu hasil yang bisa difoto.
- **Kenapa sekarang:** interaktif untuk pameran Prodi Informatika **UIN K.H. Abdurrahman Wahid Pekalongan** (**TBD:** tanggal acara).
- **Target pengguna:** pengunjung pameran, umumnya pelajar/mahasiswa/umum, level teknis awam. Dioperasikan di **laptop dengan webcam**, dijaga oleh panitia/operator.
- **Platform:** web (single-page app), **frontend saja**, berjalan **offline** di laptop pameran (PWA + aset lokal), di-host di **Vercel**.
- **Kriteria sukses (terukur):**
  1. Satu putaran pengunjung (tekan Mulai → hasil tampil) selesai dalam **≤ 10 detik**, target 5–8 detik.
  2. Aplikasi berjalan penuh **tanpa internet** setelah dimuat sekali dan di-cache (diuji dengan mode pesawat/Network: Offline).
  3. **Nol data** tersimpan: tidak ada pengiriman jaringan berisi foto/wajah, tidak ada localStorage/IndexedDB/cookie berisi data pengunjung.
  4. Berjalan stabil minimal **4 jam non-stop** tanpa refresh manual (diuji saat gladi).
  5. Tidak ada karir yang sama muncul dua kali berturut-turut pada mode default.

## 2. Scope & Constraints

**In-scope (MVP):**
- Layar sambutan + persetujuan privasi (tap untuk mulai)
- Akses webcam, deteksi wajah di perangkat (on-device), overlay titik landmark
- Prompt "Senyum dulu dong!" + deteksi senyum untuk memengaruhi teks hasil
- Animasi pemindaian + metrik analisis palsu
- Pemilihan karir acak berbobot (level kelangkaan), tanpa pengulangan berturut-turut
- Layar hasil: karir, deskripsi, statistik, badge kelangkaan, QR code
- Kartu hasil (Canvas) yang bisa difoto dari layar
- Mode tanpa kamera (cadangan)
- Mode kiosk: auto-reset, fullscreen, pintasan operator
- Offline penuh (PWA), deploy Vercel
- 50 karir + bank teks (Lampiran A & B)

**Out-of-scope / Non-Goals (sengaja tidak dibuat):**
- Backend, database, akun, login, analytics, pelacakan, formulir pendaftaran
- Menyimpan foto/wajah/hasil pengunjung dalam bentuk apa pun
- Pemindaian lebih dari satu wajah per sesi (aturan: **satu sesi satu wajah**)
- Klaim ilmiah/prediksi asli, deteksi usia/gender/emosi/etnis (dilarang)
- Efek suara (default tidak ada; **TBD** jika panitia mau)
- Dukungan multi-bahasa (hanya Bahasa Indonesia)
- Dukungan HP pengunjung sebagai perangkat utama (UI tidak boleh rusak di layar kecil, tapi bukan target desain)

**Batasan:**
- Waktu/budget: tier gratis (Vercel Hobby, library open source). **TBD:** deadline acara.
- Teknologi wajib: frontend-only, tanpa panggilan jaringan saat runtime, tanpa CDN eksternal (font, script, model, ikon semua dilayani lokal).
- Browser target: Chrome/Edge versi terbaru di laptop pameran. **TBD:** spesifikasi laptop (RAM/GPU) dan sistem operasi.

**Asumsi yang belum pasti (`TBD`):**
- Nama resmi acara/stand, tanggal acara
- Logo prodi/kampus (belum ada; sediakan placeholder yang mudah diganti)
- Palet warna resmi (belum ada; pakai palet default Bagian 9)
- URL tujuan QR code (Instagram/website prodi)
- Spesifikasi laptop, resolusi layar/proyektor
- Apakah tombol unduh kartu diaktifkan (default **nonaktif**, lihat Bagian 18)
- Apakah mode hasil "konsisten per wajah" diaktifkan (default **nonaktif**, lihat Bagian 16)

## 3. User Roles & Permission Matrix

Tidak ada autentikasi. Dua peran informal:

| Role | Fitur yang bisa diakses | Lihat data | Buat data | Ubah data | Hapus data |
|------|-------------------------|-----------|-----------|-----------|-----------|
| Pengunjung | Mulai scan, lihat hasil, coba lagi, mode tanpa kamera | Hanya hasilnya sendiri (di memori, sementara) | Tidak ada (tidak ada penyimpanan) | Tidak ada | Otomatis hilang saat reset |
| Operator (penjaga laptop) | Semua di atas + pintasan keyboard (Shift+F fullscreen, Shift+R reset paksa, Shift+N toggle mode tanpa kamera) | Sama | Tidak ada | Mengubah file konfigurasi hanya lewat kode/deploy, bukan lewat UI | Tidak ada |

> Pintasan operator memakai **Shift+huruf** agar tidak terpicu tidak sengaja oleh pengunjung yang menyentuh keyboard. Tidak ada panel admin di UI.

## 4. User Flow

```text
IDLE (layar sambutan + privasi)
  └─ tap "Mulai"
      └─ MEMINTA_KAMERA
          ├─ izin ok ─► POSISI_WAJAH (panduan oval) ─► wajah stabil ≥ 1 dtk
          │              └─ SENYUM (maks 3 dtk, lanjut otomatis meski tidak senyum)
          │                  └─ MEMINDAI (animasi 3–4 dtk, landmark + metrik palsu)
          │                      └─ HASIL (karir + kartu + QR)
          │                          ├─ "Orang berikutnya" / idle 45 dtk ─► IDLE
          │                          └─ "Coba lagi" ─► POSISI_WAJAH
          └─ izin ditolak / kamera tidak ada / model gagal ─► MODE_TANPA_KAMERA
                 (tahan tombol besar 3 dtk = "scan sidik jari" gimik) ─► MEMINDAI ─► HASIL
```

**Non-happy-path wajib ada:**
- **Loading:** model deteksi wajah sedang dimuat (indikator + teks lucu).
- **Empty/No-face:** wajah tidak terdeteksi > 6 detik → petunjuk ("Dekatkan wajah, cari tempat yang lebih terang").
- **Multi-face:** terdeteksi > 1 wajah → overlay "Satu orang dulu ya!" dan proses ditahan sampai tinggal satu wajah.
- **Error kamera:** izin ditolak / kamera tidak ditemukan / kamera dipakai aplikasi lain → pesan ramah + tombol "Coba tanpa kamera".
- **Error model:** model gagal dimuat → otomatis tawarkan mode tanpa kamera.
- **Wajah hilang di tengah scan:** jeda, kembali ke POSISI_WAJAH.
- **Idle timeout:** 45 detik tanpa interaksi di layar hasil → reset ke IDLE dan bersihkan semua data sementara.

---

# BAGIAN B — Requirements

## 5. Functional Requirements

```text
FR-001 — Layar sambutan.
Acceptance Criteria:
- Menampilkan judul acara, nama prodi + UIN K.H. Abdurrahman Wahid Pekalongan, dan tombol "Mulai" besar
- Seluruh teks Bahasa Indonesia; <html lang="id">
- Kamera TIDAK menyala sebelum pengunjung menekan "Mulai"

FR-002 — Akses kamera.
Acceptance Criteria:
- Memakai getUserMedia (kamera depan/webcam), resolusi ideal 1280x720, tanpa audio
- Jika izin ditolak/kamera tidak ada, tampil pesan ramah dan opsi mode tanpa kamera
- Stream dihentikan (track.stop()) saat kembali ke IDLE atau saat tab disembunyikan

FR-003 — Pernyataan privasi & disclaimer.
Acceptance Criteria:
- Layar sambutan menampilkan: "Kamera hanya diproses di perangkat ini. Tidak ada foto yang diunggah atau disimpan."
- Menampilkan: "Ini hanya hiburan, bukan tes bakat. Wajah tidak bisa memprediksi karir."
- Disclaimer singkat juga ada di layar hasil dan di kartu hasil

FR-004 — Deteksi wajah on-device.
Acceptance Criteria:
- Memakai MediaPipe Face Landmarker (lihat Bagian 11) yang berjalan sepenuhnya di browser
- Mendeteksi sampai 2 wajah (untuk keperluan peringatan multi-face), memproses 1 wajah
- Tidak ada pengiriman frame/gambar ke jaringan (diverifikasi via tab Network)

FR-005 — Panduan posisi wajah.
Acceptance Criteria:
- Overlay oval panduan; berubah hijau saat wajah berada di dalam oval dan stabil ≥ 1 detik
- Teks bantuan jika wajah terlalu jauh/dekat/terpotong

FR-006 — Aturan satu wajah per sesi.
Acceptance Criteria:
- Jika terdeteksi >1 wajah, proses ditahan dan tampil "Satu orang dulu ya!"
- Proses lanjut otomatis saat tinggal 1 wajah

FR-007 — Prompt senyum.
Acceptance Criteria:
- Menampilkan "Senyum dulu dong!" maksimal 3 detik
- Senyum dideteksi dari blendshape wajah (mouthSmileLeft/Right) dengan ambang yang bisa diatur di config
- Jika tidak senyum sampai timeout, lanjut dengan varian teks "netral" (tidak ada penalti/kesan negatif)

FR-008 — Animasi pemindaian.
Acceptance Criteria:
- Durasi 3–4 detik, titik/mesh landmark digambar di atas video, garis pemindai bergerak
- Teks status berganti tiap ±0,6 detik dari bank teks (Lampiran B)
- Menghormati prefers-reduced-motion (animasi dikurangi, durasi tetap)

FR-009 — Metrik analisis palsu.
Acceptance Criteria:
- Menampilkan 6 metrik (lihat Bagian 16) berupa persentase animasi hitung naik
- Dua metrik "stat unggulan" milik karir hasil selalu bernilai tinggi (85–99%), sisanya acak 25–84%
- Tampil jelas berlabel hiburan (bukan hasil analisis sungguhan)

FR-010 — Pemilihan karir acak berbobot.
Acceptance Criteria:
- Pemilihan memakai bobot kelangkaan: Umum 50%, Langka 30%, Epik 15%, Legendaris 5% (konfigurabel)
- Di dalam satu tier, pemilihan seragam (uniform)
- Karir yang sama tidak muncul dua kali berturut-turut (mode default)
- Hasil yang terakhir hanya disimpan di variabel memori, TIDAK di storage; hilang saat halaman dimuat ulang

FR-011 — Layar hasil.
Acceptance Criteria:
- Menampilkan emoji/ikon, nama karir, badge kelangkaan (warna + efek berbeda per tier), tagline, deskripsi "kamu cocok jadi … karena …", 6 statistik, teks varian senyum/netral
- Menampilkan foto snapshot pengunjung (opsional lewat config SHOW_PHOTO, default aktif) dari memori saja
- Tombol "Coba lagi" dan "Orang berikutnya", besar dan mudah ditekan

FR-012 — Level kelangkaan.
Acceptance Criteria:
- Empat tier: Umum, Langka, Epik, Legendaris
- Tier Epik dan Legendaris punya efek visual khusus (kilau/konfeti ringan), tetap ringan di laptop biasa

FR-013 — Kartu hasil.
Acceptance Criteria:
- Dibuat dengan Canvas 2D di browser, ukuran 1080x1350 (rasio 4:5)
- Berisi: foto (jika SHOW_PHOTO aktif), nama karir, badge tier, tagline, 3 statistik teratas, QR code, nama prodi + kampus, disclaimer hiburan
- Ditampilkan di layar agar bisa difoto pengunjung dengan HP-nya
- Tombol unduh hanya muncul jika config ENABLE_DOWNLOAD = true (default false)

FR-014 — QR code.
Acceptance Criteria:
- Dibuat di browser (tanpa layanan eksternal) dari URL di config (TBD: Instagram/website prodi)
- Jika URL belum diisi, QR disembunyikan (bukan menampilkan QR salah)

FR-015 — Mode tanpa kamera.
Acceptance Criteria:
- Tersedia dari layar sambutan dan otomatis ditawarkan saat kamera/model gagal
- Interaksi: tahan tombol besar 3 detik (animasi "scan sidik jari" gimik) lalu lanjut ke MEMINDAI → HASIL
- Hasil tetap dari logika pemilihan yang sama; tanpa foto di hasil/kartu

FR-016 — Mode kiosk.
Acceptance Criteria:
- Auto-reset ke IDLE setelah 45 detik tanpa interaksi di layar hasil (konfigurabel)
- Tombol fullscreen (Shift+F juga bisa), kursor disembunyikan saat idle di mode kiosk
- Tidak ada tautan keluar ke situs lain yang bisa dipencet pengunjung

FR-017 — Offline penuh.
Acceptance Criteria:
- Setelah dimuat sekali online dan di-cache, seluruh alur bekerja dengan jaringan dimatikan
- Aset model, WASM, font, ikon, dan data karir dilayani dari origin sendiri
- Ada indikator kecil di pojok untuk operator: "Siap offline ✓" saat precache selesai

FR-018 — Pembersihan data sementara.
Acceptance Criteria:
- Saat reset ke IDLE: snapshot foto, ImageBitmap/Object URL, hasil sesi, dan canvas kartu dibebaskan (revokeObjectURL, close(), clear)
- Tidak ada penulisan ke localStorage, sessionStorage, IndexedDB, Cache API (selain cache aset PWA), cookie untuk data pengunjung

FR-019 — Ketahanan jangka panjang.
Acceptance Criteria:
- Tidak ada kebocoran memori yang terlihat setelah 100 putaran berturut-turut (diuji manual dengan Task Manager Chrome)
- Loop deteksi dihentikan saat state tidak membutuhkannya

FR-020 — Aksesibilitas dasar.
Acceptance Criteria:
- Semua tombol bisa dioperasikan keyboard, fokus terlihat jelas
- Kontras teks memenuhi WCAG AA, ukuran target sentuh/klik ≥ 56px
```

## 6. Non-Functional Requirements

- **Keamanan:** frontend-only, tidak ada secret. Fokus pada CSP, tidak ada skrip eksternal, tidak ada `dangerouslySetInnerHTML`, dependency diaudit (Bagian 17).
- **Performa:** target pada laptop pameran (**TBD** spesifikasi): deteksi ≥ 15 FPS, transisi antar layar < 300 ms, waktu muat awal (online, tanpa cache) < 5 detik pada koneksi wajar, memori stabil setelah 100 putaran. Jika GPU delegate gagal, otomatis fallback ke CPU.
- **Skalabilitas:** tidak relevan (satu perangkat, satu pengguna sekali waktu).
- **Kepatuhan/Privasi:** tidak ada pengumpulan data pribadi. Biometrik tidak disimpan. Tetap tampilkan pernyataan privasi karena kamera wajah dianggap sensitif oleh pengunjung.
- **Kompatibilitas:** Chrome/Edge terbaru (desktop). Wajib diuji di laptop pameran yang sebenarnya.

## 7. Feature Catalog

| ID | Fitur | Role | Prioritas |
|----|-------|------|-----------|
| F-001 | Layar sambutan + privasi + disclaimer | Pengunjung | Must |
| F-002 | Akses kamera + deteksi wajah on-device | Pengunjung | Must |
| F-003 | Panduan posisi + aturan satu wajah | Pengunjung | Must |
| F-004 | Animasi pemindaian + landmark overlay | Pengunjung | Must |
| F-005 | Pemilihan karir acak berbobot + anti-pengulangan | Pengunjung | Must |
| F-006 | Layar hasil (karir, deskripsi, stat, varian teks) | Pengunjung | Must |
| F-007 | Metrik analisis palsu | Pengunjung | Must |
| F-008 | Level kelangkaan + efek visual | Pengunjung | Must |
| F-009 | Mode tanpa kamera (cadangan) | Pengunjung | Must |
| F-010 | Mode kiosk (auto-reset, fullscreen, pintasan) | Operator | Must |
| F-011 | Offline penuh (PWA) | Operator | Must |
| F-012 | Prompt senyum + deteksi ekspresi | Pengunjung | Should |
| F-013 | Kartu hasil (Canvas) untuk difoto | Pengunjung | Should |
| F-014 | QR code ke Instagram/website prodi | Pengunjung | Should |
| F-015 | Unduh kartu (default mati) | Pengunjung | Could |
| F-016 | Mode hasil "konsisten per wajah" (default mati) | Operator | Could |
| F-017 | Efek suara | Pengunjung | Won't (v1) |
| F-018 | Statistik penggunaan / analytics | — | Won't |

## 8. Screen / Page Specification

```text
Screen: Sambutan (IDLE)
Purpose: Mengundang pengunjung, menyampaikan privasi + disclaimer.
Components: Judul acara, logo placeholder, tombol "Mulai" (besar), tautan kecil "Coba tanpa kamera", teks privasi, indikator "Siap offline ✓" (kecil, pojok)
States: Loading model (di latar) / Siap / Error model (tawarkan mode tanpa kamera)

Screen: Posisi Wajah
Purpose: Mengarahkan wajah ke dalam oval.
Components: Video webcam (mirror), oval panduan, teks bantuan, tombol "Batal"
States: Mencari wajah / Wajah terlalu jauh / Banyak wajah / Siap (hijau)

Screen: Senyum
Purpose: Memicu senyum untuk varian teks hasil.
Components: Video, teks besar "Senyum dulu dong!", hitung mundur 3 detik
States: Senyum terdeteksi / Timeout (lanjut netral)

Screen: Memindai
Purpose: Animasi pemindaian menuju hasil.
Components: Video + mesh landmark, garis pemindai, teks status berganti, bar progres, 6 metrik naik
States: Berjalan / Wajah hilang (jeda & kembali ke Posisi Wajah)

Screen: Hasil
Purpose: Menampilkan karir, deskripsi, kartu, QR.
Components: Badge tier, emoji, nama karir, tagline, deskripsi, 6 stat bar, kartu hasil, QR, disclaimer, tombol "Coba lagi" & "Orang berikutnya"
States: Normal / Tier Epik-Legendaris (efek khusus) / Tanpa foto (mode tanpa kamera)

Screen: Mode Tanpa Kamera
Purpose: Cadangan jika kamera/model gagal.
Components: Tombol tahan 3 detik (lingkaran besar dengan cincin progres), teks gimik
States: Menunggu / Menahan / Selesai → Memindai

Overlay: Error
Purpose: Menyampaikan masalah dengan ramah.
Components: Ikon, pesan (Lampiran B), tombol aksi ("Coba lagi" / "Coba tanpa kamera")
```

---

# BAGIAN C — Desain

## 9. UI/UX Guidelines & Design System

- **Design philosophy:** playful-techy, seperti "terminal futuristik yang ramah". Energik, besar, terbaca dari jarak 1–2 meter. Satu fokus per layar.
- **Palet default (bisa diganti lewat CSS variables, belum ada palet resmi):**
  ```text
  --bg:        #0B1220   (navy gelap)
  --surface:   #121B2E
  --text:      #F1F5F9
  --muted:     #94A3B8
  --accent:    #2DD4BF   (teal)
  --accent-2:  #FBBF24   (amber)
  --danger:    #F87171
  Tier: Umum #94A3B8 | Langka #38BDF8 | Epik #A78BFA | Legendaris #FBBF24
  ```
  Semua warna wajib dipakai lewat variabel token (mudah diganti saat palet/logo resmi tersedia). Pastikan kontras AA.
- **Tipografi:** font **self-hosted** (mis. paket @fontsource) atau system font stack. **Dilarang** memuat Google Fonts/CDN (harus offline). Judul tebal besar, body ≥ 20px untuk layar kiosk.
- **Logo/slogan:** slot placeholder untuk logo prodi/kampus (file di `src/assets/brand/`, mudah diganti). Slogan *working copy* (boleh diganti panitia): "Temukan karir Informatika yang cocok denganmu!"
- **Komponen:** Button (besar, 56px+), Card, Badge tier, StatBar, ProgressRing, Toast/Overlay error, VideoStage (video + canvas overlay).
- **Layout:** utama **landscape** layar laptop (target 1280×720 hingga 1920×1080), skala memakai satuan relatif (rem/vw/clamp). Tidak rusak di portrait/layar kecil (tidak jadi target desain).
  ```text
  Kiosk landscape: ≥ 1024px (target utama)
  Tablet/portrait: 640–1024px (harus tetap terbaca)
  Kecil:           < 640px (tidak rusak, tidak dioptimalkan)
  ```
- **Dark mode:** tema gelap saja (satu tema).
- **Gerak:** animasi halus berbasis CSS/Canvas, hindari library animasi berat. Hormati `prefers-reduced-motion`.
- **Aturan konten/tone:** santai ala anak kampus, hangat, tidak menyinggung. **Dilarang** komentar tentang bentuk fisik, ras, agama, gender, disabilitas. Humor menyasar kebiasaan dunia IT (begadang, bug, deadline), bukan pengunjung.

## 10. Accessibility

- Semantic HTML (`button`, `main`, `h1`–`h2`), `lang="id"`.
- Seluruh interaksi bisa lewat keyboard; fokus terlihat.
- Teks alternatif untuk gambar/ikon bermakna; emoji dekoratif diberi `aria-hidden`.
- Status penting (mis. "Satu orang dulu ya!") memakai `aria-live="polite"`.
- Kontras AA; jangan mengandalkan warna saja (tier juga ditulis dengan teks).
- Hormati `prefers-reduced-motion`.
- Mode tanpa kamera memastikan orang yang tidak bisa/tidak mau memakai kamera tetap bisa ikut.

---

# BAGIAN D — Arsitektur Teknis

## 11. Technology Stack

```text
Build tool : Vite
Frontend   : React + TypeScript + Tailwind CSS
Deteksi    : @mediapipe/tasks-vision (Face Landmarker, output blendshapes)
QR code    : library QR generator ringan yang berjalan di browser (mis. paket "qrcode")
PWA/Offline: vite-plugin-pwa (Workbox) dengan precache aset lokal
Font       : @fontsource/* (self-hosted) atau system fonts
Testing    : Vitest + React Testing Library (logika & komponen); uji manual untuk kamera/offline
Lint/Format: ESLint + Prettier + `tsc --noEmit`
Hosting    : Vercel (static deploy, tanpa serverless function)
Backend    : TIDAK ADA
Database   : TIDAK ADA
```

**Catatan wajib untuk AI agent (anti-halusinasi):**
- Dokumen ini **sengaja tidak mengunci nomor versi**. Saat implementasi, pakai versi stabil terbaru dan **verifikasi API, nama file, dan cara memuat model langsung dari dokumentasi resmi** (MediaPipe, vite-plugin-pwa, library QR). Jangan mengarang URL model atau nama fungsi dari ingatan; jika tidak yakin, katakan dan tanyakan.
- Aset yang harus tersedia lokal (di `public/`): file WASM MediaPipe (dari paket npm), file model Face Landmarker (`.task`, diunduh sekali saat pengembangan dari sumber resmi), dan font. Setelah itu aplikasi tidak boleh meminta apa pun ke luar origin.
- File model dan WASM berukuran besar: atur batas ukuran precache Workbox (`maximumFileSizeToCacheInBytes`) agar ikut ter-cache, dan verifikasi dengan mematikan jaringan.
- Cek bahwa Vercel menyajikan `.wasm` dengan MIME type yang benar dan model tidak terblokir CSP.

## 12. System Architecture

```text
Webcam ─► <video> ─► FaceEngine (MediaPipe, di browser) ─► landmark + blendshape
                                       │
                                       ▼
                 SessionMachine (state machine: IDLE → … → HASIL)
                    │            │              │
                    ▼            ▼              ▼
            ResultPicker    StatGenerator   TextComposer
            (bobot tier,    (6 metrik       (varian senyum/
             anti-ulang)     palsu)          netral)
                    └────────────┬──────────────┘
                                 ▼
                 UI React (layar) + CardRenderer (Canvas) + QR
```

- Semua di sisi klien. **Tidak ada pemanggilan jaringan saat runtime.**
- Data karir statis di `src/data/careers.ts` (di-bundle, divalidasi saat build/test).
- Konfigurasi di `src/config/app.config.ts` (satu tempat untuk semua angka/flag).
- Service worker hanya untuk cache aset aplikasi (bukan data pengunjung).

**Struktur folder yang disarankan:**
```text
public/
  mediapipe/            (wasm + face_landmarker.task, lokal)
  icons/
src/
  assets/brand/         (logo placeholder)
  config/app.config.ts
  data/careers.ts       (50 karir, dari Lampiran A)
  data/textBank.ts      (Lampiran B)
  core/
    sessionMachine.ts   (state machine)
    resultPicker.ts     (bobot tier + anti-ulang)
    statGenerator.ts
    textComposer.ts
    faceEngine.ts       (pembungkus MediaPipe)
    cardRenderer.ts     (Canvas)
    cleanup.ts
  components/           (Screen*, VideoStage, StatBar, Badge, ProgressRing, ErrorOverlay)
  styles/
tests/
```

## 13. Database Design (ERD)

**Tidak berlaku** — tidak ada database dan tidak ada penyimpanan data pengunjung. Satu-satunya "data" adalah katalog karir statis. Skema TypeScript:

```ts
type Tier = 'umum' | 'langka' | 'epik' | 'legendaris';
type StatId = 'LOGIKA' | 'KREATIVITAS' | 'DEBUG' | 'TIM' | 'BEGADANG' | 'GOOGLING';

interface Career {
  id: string;            // unik, mis. 'L01', 'E03', 'R05', 'U12'
  title: string;         // nama karir
  emoji: string;
  tier: Tier;
  topStats: [StatId, StatId]; // dua stat unggulan (nilai tinggi)
  tagline: string;       // satu baris
  description: string;   // "Kamu cocok jadi … karena …"
}
```

Aturan validasi (diuji otomatis): 50 entri (rentang diizinkan 40–50), `id` unik, `tier` valid, `topStats` valid & berbeda, tidak ada field kosong, `description` diawali "Kamu cocok jadi".

## 14. API Specification

**Tidak berlaku** — tidak ada API/backend. Pengganti: kontrak antar modul internal.

```ts
// core/resultPicker.ts
pickCareer(careers: Career[], lastId: string | null, rng?: () => number): Career
// bobot tier dari config; tidak mengembalikan karir dengan id === lastId

// core/statGenerator.ts
generateStats(career: Career, rng?: () => number): Record<StatId, number>
// topStats: 85–99, lainnya: 25–84

// core/textComposer.ts
composeIntro(career: Career, expression: 'senyum' | 'netral', rng?: () => number): string

// core/faceEngine.ts
init(): Promise<void>            // muat model lokal; reject jika gagal
detect(video: HTMLVideoElement, ts: number): FaceFrame
// FaceFrame { faceCount: number; landmarks?: Point[]; smileScore?: number; boundingBox?: Box }

// core/cardRenderer.ts
renderCard(input: CardInput): Promise<HTMLCanvasElement>
```

Semua fungsi acak menerima `rng` opsional agar bisa diuji secara deterministik.

## 15. State Management (Frontend)

- Gunakan **state machine** sederhana (reducer + `useReducer`, atau library ringan **hanya jika benar-benar perlu**; cek dulu apakah cukup tanpa dependency baru).
- State global minimum: `phase` (IDLE | MEMINTA_KAMERA | POSISI_WAJAH | SENYUM | MEMINDAI | HASIL | TANPA_KAMERA | ERROR), `expression`, `result` (karir + stat + intro), `lastCareerId` (hanya di memori), `snapshot` (ImageBitmap/URL, hanya di memori), `error`.
- **Dilarang** menyimpan state ke localStorage/sessionStorage/IndexedDB.
- Semua timer (idle, countdown, scan) dibersihkan saat transisi/unmount.

## 16. Spesifikasi Fitur AI

- **Tidak ada AI generatif** dan tidak ada panggilan model eksternal. Semua teks hasil berasal dari data statis (Lampiran A & B), sehingga **tidak ada risiko halusinasi atau prompt injection**.
- **Satu-satunya komponen ML:** MediaPipe Face Landmarker, berjalan **on-device**, dipakai untuk: (1) mendeteksi keberadaan wajah, (2) menghitung jumlah wajah, (3) membaca skor senyum (blendshape), (4) menggambar mesh landmark. **Dilarang** memakainya untuk memperkirakan usia, gender, emosi lain, etnis, atau atribut sensitif apa pun.
- **Fallback:** jika inisialisasi GPU gagal → coba CPU; jika model gagal → mode tanpa kamera.
- **Mode hasil (config `RESULT_MODE`):**
  - `'random'` (**default**): acak berbobot tier, tanpa pengulangan berturut-turut.
  - `'face-consistent'` (opsional, default mati): hasil diturunkan dari rasio landmark yang dikuantisasi menjadi bucket lalu di-hash. **Peringatan jujur:** ini hanya **mendekati** konsisten. Landmark berubah sedikit tiap frame/pose/pencahayaan, jadi wajah yang sama bisa jatuh ke bucket berbeda. Jangan menjanjikan "pasti sama" ke pengunjung. Jika diaktifkan, sampaikan di UI hanya sebagai "cenderung sama".
- **Metrik palsu (6 stat, label tetap):**
  ```text
  LOGIKA       → "Logika & Algoritma"
  KREATIVITAS  → "Kreativitas"
  DEBUG        → "Kesabaran Debugging"
  TIM          → "Kerja Sama Tim"
  BEGADANG     → "Daya Tahan Begadang"
  GOOGLING     → "Skill Googling & Tanya AI"
  ```
- **Kelangkaan:** bobot default Umum 50 / Langka 30 / Epik 15 / Legendaris 5, semua di `app.config.ts`.

---

# BAGIAN E — Kualitas & Keamanan

## 17. Security Requirements

Karena tidak ada backend dan tidak ada login, fokusnya pada permukaan serangan sisi klien:

- Anggap input satu-satunya adalah frame kamera dan klik/ketik operator; tidak ada input teks bebas dari pengunjung.
- **Tidak ada** `dangerouslySetInnerHTML`, `eval`, atau injeksi HTML dari data.
- **CSP ketat** lewat header Vercel (`vercel.json`): sumber skrip/gaya/gambar/font/koneksi dibatasi ke `'self'`; izinkan `wasm-unsafe-eval` hanya jika dibutuhkan MediaPipe (verifikasi di dokumentasi, jangan menebak); `blob:`/`data:` hanya jika perlu (kartu/snapshot).
- **Permissions-Policy:** izinkan kamera hanya untuk origin sendiri (`camera=(self)`), nonaktifkan mikrofon, geolokasi, dll.
- Tambahan header: `X-Content-Type-Options: nosniff`, `Referrer-Policy: no-referrer`, `X-Frame-Options`/`frame-ancestors 'none'`.
- Tidak ada secret/API key (tidak dibutuhkan). Jangan pernah menambahkannya ke repo.
- Tidak ada tautan keluar yang bisa dipencet di mode kiosk (QR hanya gambar).
- `npm audit` dijalankan sebelum deploy; lockfile di-commit; tidak menambah dependency tanpa alasan.
- HTTPS (otomatis di Vercel) wajib karena getUserMedia butuh secure context (localhost juga dianggap aman).
- Aset pihak ketiga (model, wasm, font) dilayani lokal; catat sumber dan lisensi di `THIRD_PARTY.md` (**cek lisensi dari sumber resmi, jangan mengarang**).

## 18. Privacy & Data Protection

- **Data yang diproses:** frame kamera (sementara, di memori), landmark wajah (sementara), snapshot foto untuk kartu (sementara, di memori).
- **Data yang disimpan:** **tidak ada.** Tidak ada penyimpanan persisten, tidak ada pengiriman jaringan, tidak ada analytics, tidak ada cookie.
- **Retensi:** data sementara dihapus saat reset ke IDLE atau halaman ditutup (FR-018).
- **Logging:** `console.log` tidak boleh mencetak landmark/gambar. Mode debug hanya di development.
- **Catatan unduh kartu:** tombol unduh akan menyimpan **file foto pengunjung ke folder Downloads laptop pameran**, yang melanggar prinsip "tidak menyimpan data". Karena itu `ENABLE_DOWNLOAD` default **false**; kartu cukup ditampilkan untuk difoto dengan HP pengunjung. Jika panitia tetap ingin mengaktifkan, operator wajib membersihkan folder unduhan setelah acara dan pengunjung diberi tahu.
- **Pernyataan ke pengunjung:** teks privasi pada layar sambutan (FR-003).
- Jika nanti ada pihak yang bertanya soal kepatuhan (UU PDP), jawaban faktual yang bisa diberikan: data tidak dikumpulkan dan tidak meninggalkan perangkat. Ini bukan nasihat hukum; konsultasikan ke pihak kampus jika perlu.

## 19. Error Handling Standard

Tidak ada HTTP error, jadi pakai kode internal:

```text
CAM_DENIED      → izin kamera ditolak
CAM_NOT_FOUND   → kamera tidak ditemukan
CAM_BUSY        → kamera dipakai aplikasi lain
MODEL_LOAD_FAIL → model/wasm gagal dimuat
NO_FACE_TIMEOUT → wajah tidak terdeteksi > 6 detik
MULTI_FACE      → lebih dari satu wajah
UNEXPECTED      → error tak terduga (ditangkap ErrorBoundary)
```

Aturan: pesan ke pengunjung ramah dan singkat (Lampiran B), **tidak** menampilkan stack trace atau detail teknis. Setiap error punya aksi lanjutan (coba lagi / mode tanpa kamera / kembali ke awal). `ErrorBoundary` di level root agar layar tidak pernah putih kosong; jika crash, otomatis kembali ke IDLE.

## 20. Testing Strategy

- **Unit (Vitest):**
  - `resultPicker`: tidak pernah mengembalikan `lastId`; distribusi tier mendekati bobot pada sampel besar (toleransi wajar); dengan `rng` deterministik hasil bisa diprediksi.
  - `statGenerator`: `topStats` selalu 85–99, lainnya 25–84, semua 6 stat terisi.
  - `textComposer`: varian senyum/netral memuat nama karir dan tidak ada placeholder `{karir}` tersisa.
  - Validasi `careers.ts`: aturan di Bagian 13 (jumlah 40–50, id unik, tier valid, dll.).
  - `sessionMachine`: transisi valid, transisi tidak valid ditolak, timeout idle.
- **Komponen:** layar sambutan, tombol, overlay error, ProgressRing.
- **Manual (wajib, checklist Bagian 21):** kamera, deteksi senyum, multi-face, offline, kiosk 4 jam, fullscreen, laptop pameran asli.
- E2E otomatis dengan kamera sulit; **TBD** apakah perlu (mis. Playwright dengan kamera palsu). Tidak wajib untuk MVP.

## 21. Quality Gates / Definition of Done

```text
Sebuah fitur dianggap SELESAI apabila:
[ ] Requirement & acceptance criteria (FR-xxx) terpenuhi
[ ] Validasi & error handling ada (termasuk kamera ditolak/hilang)
[ ] `tsc --noEmit` dan ESLint lulus tanpa error
[ ] Test terkait lulus
[ ] Tidak ada request jaringan keluar saat runtime (cek tab Network)
[ ] Tidak ada penulisan ke localStorage/sessionStorage/IndexedDB berisi data pengunjung
[ ] Tampilan terbaca di 1280x720 dan 1920x1080, punya state loading/empty/error
[ ] Tidak ada console error/warning penting
[ ] Dokumentasi terkait diperbarui (README + Change Log)
```

**Checklist final sebelum acara (manual):**
```text
[ ] Dibuka sekali online di laptop pameran, indikator "Siap offline ✓" muncul
[ ] Jaringan dimatikan, aplikasi di-refresh, alur penuh tetap jalan
[ ] Izin kamera sudah diberikan permanen untuk origin tersebut
[ ] 100 putaran berturut-turut tanpa melambat/crash
[ ] Uji kondisi cahaya terang, redup, backlight, berkacamata, berkerudung/topi
[ ] Uji multi-face, tanpa wajah, kamera dicabut saat berjalan
[ ] Mode tanpa kamera berfungsi
[ ] Fullscreen + auto-reset 45 detik berfungsi
[ ] QR terbaca oleh beberapa HP berbeda dan mengarah ke URL yang benar
[ ] Rencana cadangan tersedia (lihat Bagian 23)
```

---

# BAGIAN F — Operasional (diringkas)

## 22. Environment Configuration
- Dua lingkungan: `development` (lokal) dan `production` (Vercel). Tidak ada `.env` rahasia; flag perilaku ada di `app.config.ts`.
- Mode debug (overlay FPS/skor senyum) hanya aktif di development.

## 23. Deployment & CI/CD
- Build statis (`vite build`) → deploy ke Vercel (hubungkan repo GitHub, auto-deploy dari `main`).
- Tambahkan `vercel.json` untuk header keamanan (Bagian 17) dan cache aset.
- CI sederhana (opsional, GitHub Actions): `tsc --noEmit`, lint, `vitest run`, `vite build`.
- **Rencana cadangan acara (wajib disiapkan):** simpan folder hasil build (`dist/`) di laptop pameran dan di flash disk, jalankan lokal lewat server statis (mis. `npx serve dist`) lalu buka `http://localhost:<port>` (kamera diizinkan di localhost). Jangan membuka `index.html` langsung via `file://` — kemungkinan besar kamera, service worker, dan WASM tidak berfungsi; **wajib diuji**, jangan diasumsikan.
- Rollback: Vercel menyimpan deployment sebelumnya; promosikan deployment lama jika versi baru bermasalah. Namun saat acara, hindari deploy baru; gunakan versi yang sudah teruji.

## 24. Observability
- Tidak ada logging jarak jauh/monitoring. Overlay debug lokal (development) saja. Tidak ada log berisi data wajah.

## 25. Documentation Requirements
```text
README.md   → cara jalan, build, deploy, cara menyiapkan laptop pameran, cara mengganti logo/warna/URL QR
THIRD_PARTY.md → daftar aset/library pihak ketiga + sumber & lisensi (diverifikasi, bukan ditebak)
CHANGELOG.md → ringkasan perubahan
```

---

# BAGIAN G — Cara AI Bekerja

## 26. AI Operating Rules

```text
1. Jangan mengarang requirement yang tidak ada di dokumen ini.
2. Jangan mengubah arsitektur tanpa alasan yang dijelaskan dan persetujuan saya.
3. Jangan menghapus fungsionalitas yang sudah ada tanpa persetujuan eksplisit.
4. Jangan menaruh secret/API key di source code (proyek ini tidak butuh satu pun).
5. Jangan menambah dependency baru tanpa alasan jelas. Cek dulu apakah sudah tersedia atau bisa tanpa library.
6. Jangan menganggap tugas selesai hanya karena build/dev server berjalan.
7. Selalu validasi input dan tangani error di setiap fitur.
8. Selalu pertimbangkan privasi dan keamanan sebelum "terlihat berfungsi".
9. Jika requirement ambigu, tandai sebagai ambiguitas dan tanyakan; bila tidak bisa, ambil keputusan paling konservatif dan laporkan.
10. Jangan mengarang URL, nama file, versi, atau API library dari ingatan. Verifikasi di dokumentasi resmi; jika tidak bisa diverifikasi, katakan "saya tidak tahu / belum terverifikasi".
11. Jangan menyimpan data pengunjung dalam bentuk apa pun (storage, jaringan, file, log).
12. Jangan menambah klaim ilmiah, deteksi usia/gender/emosi/etnis, atau komentar tentang fisik pengunjung.
13. Setelah implementasi, lakukan self-review terhadap Quality Gates (Bagian 21).
14. Di akhir setiap tahap, laporkan file yang diubah dan risiko yang tersisa.
```

## 27. AI Execution Protocol

```text
STEP 1  — Baca bagian dokumen ini yang relevan dengan tahap saat ini
STEP 2  — Periksa kode/struktur yang sudah ada
STEP 3  — Identifikasi dependency yang dibutuhkan; verifikasi di dokumentasi resmi
STEP 4  — Buat rencana implementasi singkat dan sampaikan untuk saya review
STEP 5  — Implementasi sesuai rencana yang disetujui
STEP 6  — Jalankan test/lint/typecheck yang relevan
STEP 7  — Periksa checklist keamanan & privasi (Bagian 17–18)
STEP 8  — Cocokkan dengan Quality Gates (Bagian 21)
STEP 9  — Perbaiki masalah yang ditemukan
STEP 10 — Laporkan perubahan dan hal yang masih perlu diperhatikan
```

## 28. Development Roadmap (Fase Pengerjaan)

```text
Fase 0 — Setup proyek
  Vite + React + TS + Tailwind, ESLint/Prettier, Vitest, struktur folder, app.config.ts
  Selesai jika: `npm run dev`, `tsc --noEmit`, `vitest run` jalan

Fase 1 — Data & logika murni (tanpa kamera)
  careers.ts (Lampiran A), textBank.ts (Lampiran B), resultPicker, statGenerator, textComposer + unit test
  Selesai jika: semua unit test lulus, validasi 50 karir lulus

Fase 2 — Kerangka UI & state machine (tanpa kamera)
  Layar: Sambutan, Memindai (animasi dummy), Hasil, Mode Tanpa Kamera, Error; sessionMachine; idle reset
  Selesai jika: alur penuh bisa dijalankan lewat Mode Tanpa Kamera

Fase 3 — Kamera & deteksi wajah
  faceEngine (MediaPipe lokal), VideoStage, panduan oval, aturan satu wajah, prompt senyum, penanganan error kamera
  Selesai jika: alur kamera penuh jalan; multi-face & no-face tertangani; stream dihentikan saat reset

Fase 4 — Polesan pemindaian & hasil
  Landmark overlay, metrik palsu animasi, badge/efek kelangkaan, desain final sesuai Bagian 9
  Selesai jika: tampilan final terbaca di 1280x720 & 1920x1080

Fase 5 — Kartu hasil & QR
  cardRenderer (Canvas), QR offline, flag ENABLE_DOWNLOAD & SHOW_PHOTO
  Selesai jika: kartu tampil benar, QR terbaca di beberapa HP, URL kosong → QR tersembunyi

Fase 6 — Offline (PWA) & mode kiosk
  vite-plugin-pwa + precache model/wasm/font, indikator "Siap offline", fullscreen, pintasan operator, kursor tersembunyi
  Selesai jika: lulus uji jaringan dimatikan di laptop pameran

Fase 7 — Pengujian & hardening
  Checklist Bagian 21, uji 100 putaran/4 jam, audit dependency, CSP & header, THIRD_PARTY.md
  Selesai jika: semua centang checklist final

Fase 8 — Deploy & gladi
  Deploy Vercel, uji ulang di URL produksi, siapkan paket cadangan lokal (dist/ + server statis)
  Selesai jika: gladi bersih di laptop & layar pameran sebenarnya
```

## 29. Git & Version Control Workflow

```text
Branch : main (stabil, auto-deploy), develop, feature/*, fix/*
Commit : feat:, fix:, refactor:, test:, docs:, chore:
```
- Satu fase = satu atau beberapa PR kecil; tiap PR lolos typecheck/lint/test.
- Jangan commit file model/wasm besar tanpa memeriksa batas ukuran repo/host; **TBD** (Git LFS atau skrip unduh saat setup). Jangan menebak: tanyakan ke saya.
- Tag rilis sebelum acara (mis. `v1.0-pameran`) dan jangan deploy perubahan baru ke produksi pada hari acara.

## 30. Change Log

```text
[2026-11-08] — Blueprint v1.0 dibuat — Hasil interview awal
[2026-11-08] — Perangkat: laptop + webcam; hosting Vercel; offline wajib — Jawaban pemilik proyek
[2026-11-08] — 40–50 karir; gaya santai ala anak kampus; nuansa UIN K.H. Abdurrahman Wahid Pekalongan — Jawaban pemilik proyek
[2026-11-08] — Tidak ada logo/palet resmi → palet default + placeholder logo — Belum tersedia
[2026-11-08] — Satu sesi satu wajah — Jawaban pemilik proyek
[2026-11-08] — Semua rekomendasi diterima (animasi landmark & metrik palsu, prompt senyum, kartu hasil, kelangkaan, QR, mode kiosk, privasi, mode tanpa kamera) — Jawaban pemilik proyek
[2026-11-08] — RESULT_MODE default 'random' (tanpa pengulangan berturut-turut); 'face-consistent' opsional & tidak dijamin konsisten — Rekomendasi #2 dan #9 saling berbeda; dipilih default yang paling aman. TBD konfirmasi pemilik proyek
[2026-11-08] — ENABLE_DOWNLOAD default false — Unduh menyimpan foto ke laptop pameran, bertentangan dengan "tanpa menyimpan data"
```

---

# LAMPIRAN A — Daftar 50 Karir Informatika

> Total 50 karir: **Legendaris 3, Epik 7, Langka 13, Umum 27.** Stat unggulan memakai ID: LOGIKA, KREATIVITAS, DEBUG, TIM, BEGADANG, GOOGLING.
> Semua teks bersifat hiburan, tidak memuat klaim gaji, prospek, atau fakta industri. AI agent wajib menyalin teks ini apa adanya ke `careers.ts` (boleh dirapikan ejaan), dan tidak menambah klaim faktual.

## Tier LEGENDARIS (3)

#### L01 — CTO (Chief Technology Officer) 👑
- Tier: legendaris | Stat unggulan: LOGIKA, TIM
- Tagline: Dalang di balik arah teknologi.
- Deskripsi: Kamu cocok jadi CTO karena auramu itu tipe pemimpin yang bisa melihat gambaran besar tapi tetap paham detail teknisnya. Kamu terbiasa ambil keputusan dengan kepala dingin, bahkan waktu semua orang panik karena deadline. Siap-siap saja: kalendermu bakal penuh rapat, tapi tenang, kopinya juga banyak.

#### L02 — AI Research Scientist 🧠
- Tier: legendaris | Stat unggulan: LOGIKA, KREATIVITAS
- Tagline: Pencari terobosan yang belum ada di buku.
- Deskripsi: Kamu cocok jadi AI Research Scientist karena rasa penasaranmu kayak nggak punya tombol off. Kamu gampang tenggelam di paper, eksperimen, dan pertanyaan "kok bisa ya?" sampai lupa makan siang. Begitu modelmu akhirnya jalan, rasanya mirip dapat nilai A di mata kuliah yang semua orang bilang susah.

#### L03 — Startup Founder 🚀
- Tier: legendaris | Stat unggulan: KREATIVITAS, BEGADANG
- Tagline: Berani mulai, mikir sambil jalan.
- Deskripsi: Kamu cocok jadi Startup Founder karena kamu berani mulai dulu dan beresin sambil jalan. Idemu banyak, energimu juga, dan kamu nggak gampang nyerah walau pitch-mu pernah ditolak berkali-kali. Hati-hati, niat "bikin satu fitur kecil" bisa tiba-tiba berubah jadi satu perusahaan.

## Tier EPIK (7)

#### E01 — Software Architect 🏛️
- Tier: epik | Stat unggulan: LOGIKA, TIM
- Tagline: Perapi cetak biru sistem.
- Deskripsi: Kamu cocok jadi Software Architect karena kamu suka merapikan hal-hal yang berantakan, mulai dari folder sampai alur sistem. Kamu mikir jauh ke depan: "kalau penggunanya naik seribu kali lipat, gimana?" Tim bakal sering minta pendapatmu, dan kamu bakal sering menjawab "tergantung kebutuhan".

#### E02 — Machine Learning Engineer 🤖
- Tier: epik | Stat unggulan: LOGIKA, DEBUG
- Tagline: Guru privat untuk mesin.
- Deskripsi: Kamu cocok jadi Machine Learning Engineer karena kamu sabar ngajarin mesin sampai pintar, mirip ngajarin adik ngerjain PR. Kamu nyaman dengan data, angka, dan grafik yang naik-turun. Kamu juga pasti akrab dengan momen "akurasi 99%... ternyata ada yang salah di datanya".

#### E03 — Penetration Tester (Ethical Hacker) 🕵️
- Tier: epik | Stat unggulan: KREATIVITAS, DEBUG
- Tagline: Hacker topi putih, legal dan berizin.
- Deskripsi: Kamu cocok jadi Penetration Tester karena naluri penasaranmu kuat: "kalau tombol ini ditekan sambil begini, kira-kira jebol nggak ya?" Bedanya, kamu melakukannya secara legal, dengan izin, demi membuat sistem lebih aman. Kerjamu bikin developer deg-degan di awal, tapi berterima kasih di akhir.

#### E04 — Site Reliability Engineer ⏱️
- Tier: epik | Stat unggulan: DEBUG, BEGADANG
- Tagline: Penjaga sistem agar tetap hidup.
- Deskripsi: Kamu cocok jadi Site Reliability Engineer karena kamu tetap tenang saat semua orang bilang "servernya down!". Kamu suka sistem yang stabil, monitoring yang rapi, dan tidur yang nyenyak. Kamu paham betul bahwa "jangan deploy hari Jumat" itu bukan saran, tapi hukum alam.

#### E05 — Robotics Software Engineer 🦾
- Tier: epik | Stat unggulan: LOGIKA, KREATIVITAS
- Tagline: Pengubah kode jadi gerakan nyata.
- Deskripsi: Kamu cocok jadi Robotics Software Engineer karena kamu suka melihat kode berubah jadi gerak nyata. Sabarmu tebal, soalnya robot kadang menurut, kadang tiba-tiba muter di tempat. Rasa puasnya muncul pas robot akhirnya jalan lurus tanpa nabrak meja.

#### E06 — Computer Vision Engineer 👁️
- Tier: epik | Stat unggulan: LOGIKA, KREATIVITAS
- Tagline: Pengajar komputer supaya bisa melihat.
- Deskripsi: Kamu cocok jadi Computer Vision Engineer karena kamu suka mengajari komputer "melihat". Iya, mirip scan tadi, tapi yang asli jauh lebih rumit dan butuh ketelitian tinggi. Kamu juga tahu persis bedanya "kelihatan keren di demo" dengan "tetap jalan di dunia nyata".

#### E07 — Open Source Maintainer 🌍
- Tier: epik | Stat unggulan: TIM, DEBUG
- Tagline: Penjaga komunitas kode terbuka.
- Deskripsi: Kamu cocok jadi Open Source Maintainer karena kamu suka berbagi dan sabar menghadapi banyak orang. Kamu rela membaca laporan masalah, mereview kontribusi, dan membalas dengan sopan walau pertanyaannya sudah dijawab di README. Karyamu bisa dipakai orang di berbagai tempat, dan itu keren banget.

## Tier LANGKA (13)

#### R01 — Game Developer 🎮
- Tier: langka | Stat unggulan: KREATIVITAS, BEGADANG
- Tagline: Pembuat dunia yang seru dimainkan.
- Deskripsi: Kamu cocok jadi Game Developer karena kamu kreatif dan tahan begadang demi satu bug yang bikin karakter menembus tembok. Kamu paham bahwa game seru itu campuran logika, seni, dan sedikit keisengan. Hobi main game kamu sekarang resmi naik pangkat jadi "riset".

#### R02 — AR/VR Developer 🥽
- Tier: langka | Stat unggulan: KREATIVITAS, LOGIKA
- Tagline: Arsitek dunia yang belum ada.
- Deskripsi: Kamu cocok jadi AR/VR Developer karena kamu suka membangun dunia yang belum ada. Imajinasimu liar tapi tetap bisa kamu ubah jadi kode yang jalan. Kamu pasti bangga kalau melihat orang bengong memakai headset dan itu hasil karyamu.

#### R03 — Blockchain Developer ⛓️
- Tier: langka | Stat unggulan: LOGIKA, GOOGLING
- Tagline: Penjaga kepercayaan tanpa perantara.
- Deskripsi: Kamu cocok jadi Blockchain Developer karena kamu teliti dan nggak gampang percaya sebelum ada buktinya. Kamu suka sistem dengan aturan yang jelas dan transparan. Kamu juga siap menjelaskan ke keluarga di acara kumpul bahwa "ini bukan cuma soal kripto".

#### R04 — Cybersecurity Analyst 🛡️
- Tier: langka | Stat unggulan: DEBUG, LOGIKA
- Tagline: Penjaga data dari serangan.
- Deskripsi: Kamu cocok jadi Cybersecurity Analyst karena kamu waspada tapi tetap tenang. Kamu terbiasa melihat pola janggal yang luput dari orang lain. Tugasmu menjaga data orang tetap aman, dan itu pekerjaan yang pantas dihormati.

#### R05 — Data Scientist 📊
- Tier: langka | Stat unggulan: LOGIKA, KREATIVITAS
- Tagline: Pencerita di balik tumpukan angka.
- Deskripsi: Kamu cocok jadi Data Scientist karena kamu suka mencari cerita di balik angka. Kamu nyaman dengan statistik, rapi dalam berpikir, dan nggak puas sebelum tahu "kenapa". Sebagian besar waktumu mungkin habis untuk membersihkan data, tapi hasilnya worth it.

#### R06 — Cloud Engineer ☁️
- Tier: langka | Stat unggulan: DEBUG, GOOGLING
- Tagline: Pengatur server dari awan.
- Deskripsi: Kamu cocok jadi Cloud Engineer karena kamu suka hal yang bisa diatur dari mana saja tanpa harus memegang servernya. Kamu rapi soal konfigurasi dan waspada soal tagihan. Kamu juga paham hukum awan: kalau lupa mematikan layanan, dompetlah yang menangis.

#### R07 — DevOps Engineer 🔧
- Tier: langka | Stat unggulan: TIM, DEBUG
- Tagline: Jembatan antara kode dan produksi.
- Deskripsi: Kamu cocok jadi DevOps Engineer karena kamu jago jadi jembatan antara tim yang menulis kode dan tim yang menjalankannya. Kamu suka otomatisasi: kalau sudah dikerjakan tiga kali, wajib dibikinin skrip. Pipeline berwarna hijau adalah pemandangan paling menenangkan buatmu.

#### R08 — IoT Engineer 📡
- Tier: langka | Stat unggulan: KREATIVITAS, DEBUG
- Tagline: Penghubung benda sehari-hari ke internet.
- Deskripsi: Kamu cocok jadi IoT Engineer karena kamu suka menghubungkan dunia nyata dengan dunia digital, mulai dari sensor sampai dashboard. Kamu senang utak-atik perangkat kecil dan nggak takut sama kabel. Kamu juga akrab dengan momen "tadi nyala, kok sekarang mati?".

#### R09 — Embedded Systems Engineer 🔌
- Tier: langka | Stat unggulan: LOGIKA, DEBUG
- Tagline: Otak kecil di balik perangkat pintar.
- Deskripsi: Kamu cocok jadi Embedded Systems Engineer karena kamu suka bekerja dekat dengan perangkat keras dan hemat sumber daya. Kamu teliti sampai level terkecil, dan sabar saat debugging cuma bisa lewat lampu LED yang berkedip. Banyak alat di sekitar kita bisa pintar berkat orang sepertimu.

#### R10 — NLP & Chatbot Engineer 💬
- Tier: langka | Stat unggulan: LOGIKA, KREATIVITAS
- Tagline: Pengajar mesin memahami bahasa manusia.
- Deskripsi: Kamu cocok jadi NLP & Chatbot Engineer karena kamu suka bahasa dan logika sekaligus. Kamu penasaran bagaimana mesin bisa memahami kalimat manusia, termasuk kalimat yang typo. Kamu juga siap menghadapi pengguna yang menanyakan hal-hal di luar dugaan ke chatbot.

#### R11 — Digital Forensics Analyst 🔍
- Tier: langka | Stat unggulan: DEBUG, LOGIKA
- Tagline: Detektif jejak digital.
- Deskripsi: Kamu cocok jadi Digital Forensics Analyst karena kamu teliti dan suka memecahkan teka-teki dari jejak digital. Kamu tenang, sistematis, dan nggak gampang menyimpulkan tanpa bukti. Mirip detektif, tapi senjatamu laptop dan kesabaran.

#### R12 — Bioinformatics Specialist 🧬
- Tier: langka | Stat unggulan: LOGIKA, GOOGLING
- Tagline: Penerjemah kode untuk ilmu hayati.
- Deskripsi: Kamu cocok jadi Bioinformatics Specialist karena kamu bisa menjembatani dua dunia: kode dan ilmu hayati. Kamu suka pola, data besar, dan merasa bangga kalau programmu membantu penelitian. Memang harus belajar dua bidang sekaligus, tapi kamu pasti sanggup.

#### R13 — Creative Technologist ✨
- Tier: langka | Stat unggulan: KREATIVITAS, GOOGLING
- Tagline: Pencampur seni dan teknologi.
- Deskripsi: Kamu cocok jadi Creative Technologist karena kamu suka mencampur seni dan teknologi jadi sesuatu yang bikin orang bilang "wah". Kamu cepat mencoba hal baru dan nggak malu bereksperimen. Instalasi interaktif seperti yang kamu lihat di pameran ini bisa jadi makananmu sehari-hari.

## Tier UMUM (27)

#### U01 — Backend Developer 🛠️
- Tier: umum | Stat unggulan: LOGIKA, DEBUG
- Tagline: Penjaga dapur digital.
- Deskripsi: Kamu cocok jadi Backend Developer karena kamu lebih senang kalau sistem jalan lancar daripada dapat tepuk tangan. Kepalamu penuh logika, dan kamu betah mengurus hal-hal di balik layar. Bersiaplah jadi orang pertama yang dicari tiap muncul "error 500".

#### U02 — Frontend Developer 🎨
- Tier: umum | Stat unggulan: KREATIVITAS, DEBUG
- Tagline: Perapi tampilan yang dilihat semua orang.
- Deskripsi: Kamu cocok jadi Frontend Developer karena kamu peduli pada hal-hal yang langsung dilihat dan dirasakan pengguna. Kamu teliti sampai selisih satu piksel dan peka kalau tombol terasa kurang pas. Kamu juga sudah akrab dengan misteri "kenapa CSS-nya nggak mau rata tengah".

#### U03 — Full-Stack Developer 🧩
- Tier: umum | Stat unggulan: LOGIKA, GOOGLING
- Tagline: Bisa semua, dari layar sampai database.
- Deskripsi: Kamu cocok jadi Full-Stack Developer karena kamu suka mengerjakan semuanya sendiri, dari tampilan sampai database. Kamu cepat belajar dan nggak gampang bingung pindah-pindah topik. Risikonya cuma satu: kamu bakal sering diminta "sekalian bantuin yang itu ya".

#### U04 — Mobile App Developer 📱
- Tier: umum | Stat unggulan: KREATIVITAS, LOGIKA
- Tagline: Pembuat aplikasi dalam genggaman.
- Deskripsi: Kamu cocok jadi Mobile App Developer karena kamu suka membuat sesuatu yang dipakai orang setiap hari lewat genggaman. Kamu peka soal pengalaman pengguna di layar kecil dan performa di berbagai jenis HP. Rasa senangmu muncul saat melihat orang asing memakai aplikasimu.

#### U05 — UI/UX Designer 🖌️
- Tier: umum | Stat unggulan: KREATIVITAS, TIM
- Tagline: Perancang pengalaman yang enak dipakai.
- Deskripsi: Kamu cocok jadi UI/UX Designer karena kamu peka terhadap hal-hal yang bikin pengguna bingung atau senang. Kamu suka rapi, estetik, dan berpikir dari sudut pandang orang lain. Kamu juga akrab dengan revisi "bisa tolong tombolnya dibesarkan sedikit?".

#### U06 — UX Researcher 🗣️
- Tier: umum | Stat unggulan: TIM, GOOGLING
- Tagline: Pendengar kebiasaan pengguna.
- Deskripsi: Kamu cocok jadi UX Researcher karena kamu pendengar yang baik dan penasaran dengan kebiasaan orang. Kamu suka bertanya "kenapa" dengan sopan sampai ketemu akar masalahnya. Temuanmu sering bikin tim sadar bahwa pengguna nggak selalu memakai aplikasi seperti yang mereka bayangkan.

#### U07 — Data Analyst 📈
- Tier: umum | Stat unggulan: LOGIKA, GOOGLING
- Tagline: Penerjemah angka jadi cerita.
- Deskripsi: Kamu cocok jadi Data Analyst karena kamu suka merapikan angka berantakan jadi grafik yang gampang dipahami. Kamu teliti, sabar, dan senang kalau datanya akhirnya "bicara". Bonus: kamu bakal jago spreadsheet dan sering dipanggil kalau ada rumus yang error.

#### U08 — Data Engineer 🚰
- Tier: umum | Stat unggulan: LOGIKA, DEBUG
- Tagline: Pembuat saluran data.
- Deskripsi: Kamu cocok jadi Data Engineer karena kamu suka membangun saluran supaya data mengalir rapi dari sumber ke tujuan. Kamu tahan dengan pekerjaan di balik layar yang jarang disorot tapi penting banget. Para data scientist akan berterima kasih padamu, mungkin dengan suara pelan.

#### U09 — AI Application Engineer 🧰
- Tier: umum | Stat unggulan: GOOGLING, KREATIVITAS
- Tagline: Peramu AI jadi aplikasi.
- Deskripsi: Kamu cocok jadi AI Application Engineer karena kamu suka memanfaatkan teknologi AI jadi aplikasi yang benar-benar dipakai orang. Kamu cepat menangkap tren dan pintar menyusun instruksi supaya hasilnya sesuai. Kamu juga tahu bahwa AI itu alat bantu, bukan pengganti otak sendiri.

#### U10 — Network Engineer 🌐
- Tier: umum | Stat unggulan: DEBUG, TIM
- Tagline: Pengatur jalur data.
- Deskripsi: Kamu cocok jadi Network Engineer karena kamu sabar menghadapi kabel, router, dan keluhan "internetnya lemot". Kamu suka sistem yang tertata dan bisa menelusuri masalah selangkah demi selangkah. Kamu adalah alasan wifi di tempat acara bisa menyala.

#### U11 — Database Administrator 🗄️
- Tier: umum | Stat unggulan: LOGIKA, DEBUG
- Tagline: Penjaga gudang data.
- Deskripsi: Kamu cocok jadi Database Administrator karena kamu teliti dan paling nggak tahan melihat data berantakan. Kamu menjaga data supaya tertib, cepat diakses, dan aman. Kamu juga paham kenapa "backup dulu sebelum hapus" itu prinsip hidup.

#### U12 — System Analyst 📝
- Tier: umum | Stat unggulan: LOGIKA, TIM
- Tagline: Penerjemah kebutuhan jadi rancangan.
- Deskripsi: Kamu cocok jadi System Analyst karena kamu jago menerjemahkan keinginan orang yang kadang abstrak menjadi kebutuhan jelas untuk tim teknis. Kamu sabar mendengar, runtut menjelaskan, dan teliti dengan detail. Kamu juga bakal sering dengar "pokoknya yang simpel aja", lalu ditambah dua puluh fitur.

#### U13 — Business Analyst 🤝
- Tier: umum | Stat unggulan: TIM, LOGIKA
- Tagline: Penghubung bisnis dan teknologi.
- Deskripsi: Kamu cocok jadi Business Analyst karena kamu bisa ngobrol dengan orang bisnis dan orang teknis, dan dua-duanya paham. Kamu suka mencari solusi yang masuk akal dan berdampak. Kamu juga tahu bahwa teknologi canggih nggak berguna kalau nggak menyelesaikan masalah nyata.

#### U14 — Product Manager 🧭
- Tier: umum | Stat unggulan: TIM, KREATIVITAS
- Tagline: Penentu arah produk.
- Deskripsi: Kamu cocok jadi Product Manager karena kamu suka memikirkan "produk ini sebenarnya buat siapa dan buat apa?". Kamu pandai menyusun prioritas dan menyatukan banyak pendapat. Hari-harimu penuh keputusan, dan kamu bakal sering bilang "kita simpan dulu untuk versi berikutnya".

#### U15 — Project Manager / Scrum Master 📅
- Tier: umum | Stat unggulan: TIM, BEGADANG
- Tagline: Penjaga ritme tim.
- Deskripsi: Kamu cocok jadi Project Manager atau Scrum Master karena kamu jago mengatur orang dan jadwal tanpa bikin suasana tegang. Kamu rapi, komunikatif, dan peka kalau ada anggota tim yang mulai kewalahan. Kamu juga sudah kenyang pengalaman kerja kelompok di mana semua orang baru sibuk sehari sebelum deadline.

#### U16 — QA Engineer / Software Tester 🐞
- Tier: umum | Stat unggulan: DEBUG, LOGIKA
- Tagline: Pemburu bug.
- Deskripsi: Kamu cocok jadi QA Engineer karena kamu punya mata jeli untuk menemukan hal yang luput dari orang lain. Kamu suka mencoba cara-cara aneh memakai aplikasi, dan menemukan bug itu kepuasan tersendiri. Kamu menjaga kualitas supaya pengguna bukan pihak pertama yang menemukan masalah.

#### U17 — Technical Writer ✍️
- Tier: umum | Stat unggulan: KREATIVITAS, TIM
- Tagline: Penjelas hal rumit jadi mudah.
- Deskripsi: Kamu cocok jadi Technical Writer karena kamu bisa menjelaskan hal rumit dengan bahasa yang enak dibaca. Kamu sabar, runtut, dan peduli pada orang yang baru belajar. Dokumentasi yang jelas itu penyelamat, dan kamu penyelamatnya.

#### U18 — IT Consultant 💼
- Tier: umum | Stat unggulan: TIM, GOOGLING
- Tagline: Pemberi saran teknologi.
- Deskripsi: Kamu cocok jadi IT Consultant karena kamu pandai membaca masalah dan memberi saran yang masuk akal. Kamu komunikatif, mudah akrab, dan nggak gugup di depan klien. Jawaban andalanmu "tergantung konteksnya", dan kali ini itu memang benar.

#### U19 — IT Auditor 🧾
- Tier: umum | Stat unggulan: LOGIKA, DEBUG
- Tagline: Pemeriksa kepatuhan sistem.
- Deskripsi: Kamu cocok jadi IT Auditor karena kamu teliti, jujur, dan nggak gampang ditipu angka yang kelihatannya rapi. Kamu suka memastikan sistem berjalan sesuai aturan. Orang mungkin sedikit deg-degan saat kamu datang, tapi mereka tahu kerjamu membuat semuanya lebih baik.

#### U20 — IT Support Specialist 🎧
- Tier: umum | Stat unggulan: TIM, GOOGLING
- Tagline: Penolong pertama saat komputer ngambek.
- Deskripsi: Kamu cocok jadi IT Support Specialist karena kamu sabar dan suka menolong. Kamu terbiasa menenangkan orang yang panik karena "laptopnya nggak mau nyala". Kamu juga tahu bahwa kadang solusinya memang cuma restart, dan itu bukan sulap, itu ilmu.

#### U21 — Dosen / Peneliti Informatika 🎓
- Tier: umum | Stat unggulan: LOGIKA, TIM
- Tagline: Pembagi ilmu dan penjelajah riset.
- Deskripsi: Kamu cocok jadi Dosen atau Peneliti Informatika karena kamu suka belajar dan lebih suka lagi membagikan ilmunya. Kamu sabar menjelaskan, penasaran pada hal baru, dan nyaman dengan dunia riset. Kamu juga sudah terlatih menghadapi revisi dari pembimbing.

#### U22 — Content Creator Teknologi 🎬
- Tier: umum | Stat unggulan: KREATIVITAS, GOOGLING
- Tagline: Penjelas teknologi yang asyik ditonton.
- Deskripsi: Kamu cocok jadi Content Creator Teknologi karena kamu bisa menjelaskan hal teknis dengan gaya yang bikin orang betah menonton. Kamu kreatif, ekspresif, dan peka terhadap tren. Bonusnya, kamu pasti tahu perangkat mana yang layak dibeli dan mana yang cuma lucu di iklan.

#### U23 — Developer Relations 🎤
- Tier: umum | Stat unggulan: TIM, KREATIVITAS
- Tagline: Wajah ramah komunitas developer.
- Deskripsi: Kamu cocok jadi Developer Relations karena kamu mudah akrab dengan banyak orang dan suka berbagi pengalaman teknis. Kamu nyaman bicara di depan komunitas, jago bikin demo, dan sabar menjawab pertanyaan. Kamu jadi wajah teknologi yang ramah bagi para developer.

#### U24 — Developer Edukasi & Keislaman Digital 📚
- Tier: umum | Stat unggulan: KREATIVITAS, TIM
- Tagline: Pembuat teknologi untuk belajar dan berilmu.
- Deskripsi: Kamu cocok jadi Developer Edukasi & Keislaman Digital karena kamu ingin teknologi memberi manfaat bagi banyak orang, dari belajar online sampai aplikasi pendukung pembelajaran keislaman. Kamu perhatian pada isi, bukan cuma tampilan. Kamu juga paham bahwa konten yang benar dan terpercaya sama pentingnya dengan kode yang rapi.

#### U25 — Data Protection Officer 🔐
- Tier: umum | Stat unggulan: LOGIKA, TIM
- Tagline: Penjaga privasi data.
- Deskripsi: Kamu cocok jadi Data Protection Officer karena kamu peduli pada privasi dan nggak rela data orang disalahgunakan. Kamu teliti membaca aturan dan tegas tapi tetap sopan. Kamu juga satu-satunya orang di rapat yang bertanya "datanya disimpan di mana, dan boleh nggak?".

#### U26 — GIS Developer 🗺️
- Tier: umum | Stat unggulan: KREATIVITAS, LOGIKA
- Tagline: Pembaca cerita dari peta.
- Deskripsi: Kamu cocok jadi GIS Developer karena kamu suka peta dan data yang punya lokasi. Kamu senang melihat titik-titik di peta bercerita soal wisata, rute terbaik, sampai wilayah rawan bencana. Kamu nggak gampang nyasar, baik di jalan maupun di database.

#### U27 — Developer E-Government / Smart City 🏙️
- Tier: umum | Stat unggulan: TIM, DEBUG
- Tagline: Pemudah urusan layanan publik.
- Deskripsi: Kamu cocok jadi Developer E-Government / Smart City karena kamu ingin teknologi memudahkan urusan banyak orang, dari layanan publik sampai data kota. Kamu sabar dengan proses dan aturan yang berlapis. Kalau antrean pengurusan dokumen jadi berkurang karena sistemmu, kamu boleh bangga banget.

---

# LAMPIRAN B — Bank Teks

> Semua teks Bahasa Indonesia, santai, aman, tidak menyinggung. AI agent menyimpannya di `textBank.ts` dan tidak menambah teks yang menyinggung fisik/ras/agama/gender/disabilitas.

**Judul & sapaan (working copy, boleh diganti panitia):**
- Judul: "Tebak Karir Informatika"
- Sub: "Scan wajahmu, temukan karir yang cocok denganmu!"
- Tombol: "Mulai", "Coba tanpa kamera", "Coba lagi", "Orang berikutnya", "Batal"

**Privasi & disclaimer:**
- Privasi: "Kamera hanya diproses di perangkat ini. Tidak ada foto yang diunggah atau disimpan."
- Disclaimer: "Ini cuma hiburan, bukan tes bakat. Wajah nggak bisa memprediksi karir, tapi semoga hasilnya bikin kamu penasaran!"
- Disclaimer kartu (pendek): "Hanya hiburan 😄"

**Pesan status saat memindai (acak urutan, ±0,6 dtk per pesan):**
1. Memindai kontur wajah…
2. Mendeteksi aura ngoding…
3. Mengukur level begadang…
4. Menghitung kesabaran debugging…
5. Mencocokkan dengan 50 jalur karir Informatika…
6. Menanyakan pendapat server imajiner…
7. Mengompilasi bakat tersembunyi…
8. Hampir selesai, tahan ekspresimu sebentar…

**Teks panduan:**
- Posisi: "Posisikan wajahmu di dalam oval ya!"
- Terlalu jauh: "Dekatkan wajahmu sedikit."
- Terlalu dekat: "Mundur sedikit, biar kelihatan semua."
- Tidak ada wajah: "Wajahmu belum kelihatan. Coba cari tempat yang lebih terang."
- Banyak wajah: "Satu orang dulu ya! Yang lain antre dengan sabar 😄"
- Senyum: "Senyum dulu dong!"
- Wajah hilang saat scan: "Eh, wajahnya hilang. Yuk ulangi dari posisi tadi."

**Intro hasil — varian SENYUM** (pilih acak, `{karir}` diganti nama karir):
1. "Senyummu barusan terdeteksi, dan itu mengubah segalanya. Hasil analisis: kamu cocok jadi {karir}!"
2. "Senyum begitu cerah, sistem sampai terpesona. Karir yang paling cocok untukmu: {karir}!"
3. "Energi positifmu tertangkap sempurna. Hasilnya: {karir}!"
4. "Senyum lebar, vibes bagus. Sistem menyimpulkan kamu cocok jadi {karir}!"
5. "Wah, ekspresi ceriamu bikin algoritma semangat. Jawabannya: {karir}!"

**Intro hasil — varian NETRAL** (dipakai jika tidak tersenyum, tidak boleh bernada negatif):
1. "Ekspresi tenangmu menandakan orang yang fokus. Hasil analisis: kamu cocok jadi {karir}!"
2. "Wajah seriusmu terbaca sebagai kepala dingin saat deadline. Karir untukmu: {karir}!"
3. "Tatapan fokus itu pertanda calon profesional. Hasilnya: {karir}!"
4. "Aura tenang tertangkap jelas. Sistem menyimpulkan kamu cocok jadi {karir}!"
5. "Santai tapi mantap. Jawabannya: {karir}!"

**Intro mode tanpa kamera:**
- "Sidik jari imajinarimu sudah dipindai. Hasilnya: kamu cocok jadi {karir}!"
- "Tanpa kamera pun bakatmu tetap terdeteksi. Hasil: {karir}!"

**Label kelangkaan:**
- Umum: "Karir Umum" · Langka: "✨ Karir Langka!" · Epik: "💜 Karir EPIK!" · Legendaris: "👑 KARIR LEGENDARIS!"
- Teks tambahan Legendaris: "Cuma sekitar 5% pengunjung yang dapat ini. Beruntung banget!" (angka mengikuti bobot di config; jangan hardcode jika bobot diubah)

**Pesan error (ramah):**
- CAM_DENIED: "Kameranya belum diizinkan. Tenang, kamu tetap bisa ikut tanpa kamera!"
- CAM_NOT_FOUND: "Kameranya nggak ditemukan. Yuk coba tanpa kamera."
- CAM_BUSY: "Kameranya lagi dipakai aplikasi lain. Tutup dulu aplikasinya, atau coba tanpa kamera."
- MODEL_LOAD_FAIL: "Sistem pemindai lagi ngambek. Coba tanpa kamera dulu ya!"
- NO_FACE_TIMEOUT: "Wajahmu belum terdeteksi. Coba lagi atau pilih tanpa kamera."
- UNEXPECTED: "Ups, ada yang error. Kita mulai dari awal ya!"

**Mode tanpa kamera:**
- Instruksi: "Tahan tombol ini selama 3 detik untuk memindai sidik jari imajinarimu."
- Saat menahan: "Terus tahan… jangan dilepas!"

**Indikator operator:** "Siap offline ✓" / "Menyiapkan mode offline…" / "Belum siap offline" (diganti agar tetap ringkas dan kecil)

---

# LAMPIRAN C — Prompt Siap Pakai untuk AI Agent

**Prompt pembuka sesi (tempel setelah upload blueprint.md):**
```text
Saya melampirkan blueprint.md untuk proyek "Tebak Karir Informatika".
Jadikan dokumen ini sumber kebenaran utama. Jika ada bagian yang tidak jelas, tanyakan sebelum menulis kode.
Aturan: ikuti Bagian 26 (AI Operating Rules) dan 27 (Execution Protocol). Jangan mengarang URL/nama file/versi library; verifikasi di dokumentasi resmi.
Kerjakan bertahap sesuai Roadmap (Bagian 28). Mulai dari Fase 0 saja: sampaikan rencana implementasinya untuk saya review sebelum menulis kode.
```

**Prompt per fase (contoh Fase 1):**
```text
Lanjut Fase 1 (Data & logika murni). Salin Lampiran A ke src/data/careers.ts dan Lampiran B ke src/data/textBank.ts sesuai skema Bagian 13.
Implementasikan resultPicker, statGenerator, textComposer beserta unit test (Bagian 20).
Laporkan file yang diubah, hasil test, dan risiko yang tersisa. Jangan lanjut ke Fase 2 sebelum saya setujui.
```

**Prompt review keamanan & privasi (jalankan setelah Fase 6):**
```text
Lakukan review terhadap Bagian 17, 18, dan 21. Periksa: apakah ada request jaringan keluar saat runtime, penulisan ke storage berisi data pengunjung, skrip/font/aset eksternal, console.log berisi data wajah, dan kebocoran memori setelah banyak putaran. Laporkan temuan dengan lokasi file dan saran perbaikan.
```
