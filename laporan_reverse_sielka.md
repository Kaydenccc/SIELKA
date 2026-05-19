# Laporan Reverse Engineering — `libapp.so` (SIELKA v2)

## Informasi File
| Atribut | Detail |
|---|---|
| Nama file | `libapp.so` |
| Ukuran | 6.4 MB |
| Format | ELF 64-bit LSB Shared Object |
| Arsitektur | ARM AArch64 (Android) |
| Status | Stripped (simbol debug dihapus) |
| Build ID | `7a3ef930df7e79e621a2e1cc84408157` |
| Framework | Flutter / Dart (Dart VM Snapshot) |

---

## Identitas Aplikasi

| Field | Nilai |
|---|---|
| Nama Aplikasi | **SIELKA v2** |
| Kepanjangan | Sistem Informasi Elektronik Laporan Kinerja ASN Real Time Real Location |
| Instansi | Kantor Kementerian Agama Kabupaten Tana Toraja |
| App ID (Play Store) | `id.go.kemenag.sielka` |
| Copyright | 2026 Kemenag Tana Toraja |
| Play Store URL | `https://play.google.com/store/apps/details?id=id.go.kemenag.sielka` |
| Source Dev Path | `D:/projek/sielka_v2/` |

---

## API & Endpoint Server

### Base URL
```
https://absensi.kemenagtanatoraja.id
https://absensi.kemenagtanatoraja.id/api
```

### Daftar Endpoint
| Endpoint | Fungsi |
|---|---|
| `POST /api/login` | Login pengguna |
| `GET /api/me` | Profil pengguna saat ini |
| `GET /api/dashboard` | Data dashboard |
| `GET /api/app/version` | Cek versi aplikasi |
| `POST /attendance/record` | Rekam absensi |
| `GET /attendance/today` | Status absensi hari ini |
| `GET /attendance/recap` | Rekap absensi |
| `GET /lkh` | Daftar LKH |
| `POST /lkh` | Simpan LKH baru |
| `PUT /lkh/{id}` | Update LKH |
| `DELETE /lkh/{id}` | Hapus LKH |
| `POST /lkh/ai-rapikan` | Rapikan teks LKH dengan AI |
| `GET /lkh/ai-status` | Cek kuota AI |

---

## HTTP Headers Kustom

```
Authorization: Bearer <token>
X-User-NIP: <nip_pegawai>
X-Device-ID: <device_id>
User-Agent: <flutter_user_agent>
```

---

## Struktur Layar / Screen

| Screen | Deskripsi |
|---|---|
| `SplashScreen` | Layar pembuka |
| `LoginScreen` | Login (NIP + Password) |
| `HomeScreen` | Halaman utama |
| `DashboardScreen` | Dashboard ringkasan |
| `AttendanceScreen` | Input absensi |
| `AttendanceHistoryScreen` | Riwayat absensi |
| `LkhInputScreen` | Input LKH harian |
| `LkhHistoryScreen` | Riwayat LKH |
| `DeveloperOptionsBlockScreen` | Blokir jika mode developer aktif |
| `EasterEggScreen` | Easter egg tersembunyi |

---

## Struktur Source Code (Flutter/Dart)

```
sielka_v2/
├── main.dart
├── app_navigator.dart
├── app_theme_controller.dart
├── screens/
│   ├── splash_screen.dart
│   ├── login_screen.dart
│   ├── home_screen.dart
│   ├── dashboard_screen.dart
│   ├── attendance_screen.dart
│   ├── attendance_history_screen.dart
│   ├── lkh_input_screen.dart
│   ├── lkh_history_screen.dart
│   ├── developer_options_block_screen.dart
│   └── easter_egg_screen.dart
├── services/
│   ├── mobile_api_service.dart
│   ├── app_update_service.dart
│   ├── device_identity_service.dart
│   ├── device_security_service.dart
│   └── lkh_reminder_service.dart
└── widgets/
    └── location_preview_card.dart
```

---

## Data yang Disimpan Lokal (SharedPreferences)

| Key | Deskripsi |
|---|---|
| `absen_masuk` | Data absensi masuk |
| `absen_pulang` | Data absensi pulang |
| `lokasi` | Data lokasi terakhir |
| `lokasi_kerja` | Lokasi kantor/unit |
| `sumber_lokasi_kerja` | Sumber lokasi kerja (unit/khusus) |
| `is_valid_lokasi` | Status validasi lokasi |
| `lkh_hari_ini` | Cache LKH hari ini |
| `lkh_history_cache_json_` | Cache JSON riwayat LKH |
| `lkh_history_cache_saved_at_` | Timestamp cache riwayat LKH |
| `lkh_history_dirty` | Flag dirty cache LKH |
| `lkh_reminder_channel` | Notifikasi pengingat LKH |
| `lkh_reminder_diagnostics` | Diagnostik pengingat |
| `lkh_reminder_catch_up_shown_` | Flag reminder catch-up |
| `total_hadir_bulan_ini` | Total kehadiran bulan ini |
| `total_lkh_bulan_ini` | Total LKH bulan ini |
| `open_lkh` | Flag buka layar LKH |

---

## Fitur Keamanan

- **Anti-Developer Mode** — Deteksi mode pengembang Android aktif; absensi & LKH diblokir
- **Anti-FakeGPS** — Deteksi aplikasi GPS palsu:
  > *"Lokasi palsu terdeteksi. Matikan aplikasi FakeGPS lalu coba lagi."*
- **Device Binding** — Akun terikat ke satu perangkat:
  > *"Akun ini sudah terdaftar di perangkat lain. Hubungi admin jika perlu reset perangkat."*
- **GPS Accuracy Check** — Validasi akurasi lokasi GPS sebelum absensi
- **Server Time** — Waktu absensi menggunakan waktu server, bukan waktu lokal perangkat
- **SSL/TLS** — Koneksi HTTPS dengan verifikasi sertifikat server
- **Session Token** — Autentikasi berbasis Bearer token

---

## Fitur AI (Integrasi LLM)

Aplikasi memiliki fitur **"Rapikan Teks LKH dengan AI"**:

- Endpoint: `POST /lkh/ai-rapikan`
- Status kuota: `GET /lkh/ai-status`
- Kuota harian per pengguna (`ai_limit`, `ai_remaining_today`, `ai_used_today`)
- Pesan terkait:
  - *"Tulis draf singkat, lalu AI akan merapikannya menjadi kalimat LKH yang lebih formal."*
  - *"Jatah AI hari ini sudah habis."*
  - *"Teks LKH berhasil dirapikan."*
  - *"AI tidak mengembalikan teks yang bisa dipakai."*

---

## Pesan UI Penting

### Login
- `NIP atau password tidak sesuai.`
- `Login gagal. Silakan coba lagi.`
- `Terjadi kesalahan saat login. Silakan coba lagi.`
- `Balasan server tidak valid. Periksa API login.`

### Absensi
- `Absen masuk sudah tercatat. Setelah jam 11.00, isi absen akan dicatat sebagai absen pulang.`
- `Absensi berhasil dicatat.`
- `Absensi hari ini sudah lengkap. Jika perlu koreksi lokasi atau waktu pada rentang sekarang, tekan tombol untuk absen ulang.`
- `Absensi masuk dan pulang hari ini sudah tercatat.`

### Lokasi
- `GPS belum aktif. Nyalakan lokasi perangkat lalu coba lagi.`
- `Lokasi palsu terdeteksi. Matikan aplikasi FakeGPS lalu coba lagi.`
- `Lokasi Anda sekitar [X] m dari lokasi kerja. Absensi tetap bisa dilanjutkan untuk tugas di luar kantor. Lanjutkan?`
- `Izin lokasi diperlukan untuk melakukan absensi.`
- `Acuan: lokasi unit` / `Acuan: lokasi kerja khusus`

### LKH
- `LKH berhasil disimpan.`
- `LKH berhasil diperbarui.`
- `LKH berhasil dihapus.`
- `Anda hanya dapat mengubah deskripsi, jenis hasil, dan jumlah. Jam dan lokasi tidak berubah.`
- `Tanggal, jam, dan lokasi dicatat otomatis saat LKH disimpan.`

### Koneksi / Error Server
- `Koneksi terputus. Mencoba sekali lagi...`
- `Server lambat merespons. Mencoba sekali lagi...`
- `Server sedang bermasalah. Coba lagi beberapa saat.`
- `Tidak dapat terhubung ke server. Periksa koneksi internet.`
- `DNS gagal menemukan host server.`

---

## Aset & Media

- `assets/images/sielka_logo.png` — Logo aplikasi
- Database lokal: SQLite (`sqflite`) dengan tabel `cacheObject`

---

## Data Contoh / Hardcoded

Beberapa nama pegawai hardcoded (kemungkinan untuk kredit/about):
- `Dessy Malindo - MIN 4 Tana Toraja`
- `Masita Pakata - MIN 1 Tana Toraja`
- `Maynanda Restu Buana - MIN 3 Tana Toraja`
- `Muh. Asyaraf Alif Fikri - KUA Kec. Bituang`
- `Kepala Kantor Kementerian Agama Kabupaten Tana Toraja`

---

## Dependency / Package Pihak Ketiga

| Package | Fungsi |
|---|---|
| `geolocator` | Akses GPS / lokasi |
| `local_notifications` | Notifikasi lokal (pengingat LKH) |
| `cached_network_image` | Cache gambar dari network |
| `sqflite` | Database SQLite lokal |
| `shared_preferences` | Penyimpanan key-value lokal |
| `package_info` | Info versi aplikasi |
| `device_info` | Info perangkat |
| `url_launcher` | Buka URL di browser |
| `http` | HTTP client |
| `path_provider` | Akses path direktori |

---

*Laporan dihasilkan dari analisis strings & metadata ELF binary `libapp.so` (Flutter Dart snapshot)*
