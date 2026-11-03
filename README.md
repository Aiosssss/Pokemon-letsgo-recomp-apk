# ⚡ LetsGoRecomp Android







Port Android berbasis **Rekompilasi Statis Native ARM64** (*Static Recompilation*) untuk **Pokémon: Let's Go, Pikachu!** yang dibangun di atas teknologi [letsgo-recomp](https://github.com/raulzrrs/letsgo-recomp) (`LAS Recomp`) dan **SDL3 + Vulkan 1.3**. 

Berbeda dengan emulator Nintendo Switch konvensional yang menerjemahkan instruksi CPU secara dinamis saat permainan berjalan, aplikasi ini menjalankan kode biner game yang telah direkompilasi secara statis langsung di prosesor **ARM64** perangkat Android Anda. Versi `1.0.9-indonesia` ini telah dilokalisasi sepenuhnya ke dalam **Bahasa Indonesia** pada menu peluncur, pengaturan grafis, pengelolaan driver GPU, hingga konfigurasi **Joy-Con Eden Overlay**.

> **Catatan Penting (Legalitas):** File APK ini (`54.4 MB`) **tidak menyertakan aset atau ROM game milik Nintendo**. Anda wajib memiliki dan mengimpor file game resmi milik Anda sendiri (`.nsp` / `.xci`) beserta kunci dekripsi (`prod.keys`) melalui menu impor di dalam aplikasi.

## 📱 Laporan Pengujian Perangkat (POCO X7 Pro)

Pengujian dilakukan secara menyeluruh pada perangkat berarsitektur GPU **Arm Mali** untuk memvalidasi kompatibilitas *Driver Sistem Vulkan Universal* di luar ekosistem Snapdragon/Adreno:

| Parameter Pengujian | Detail Spesifikasi & Hasil |
| :--- | :--- |
| **Perangkat Uji** | [Xiaomi POCO X7 Pro](https://www.gsmarena.com/xiaomi_poco_x7_pro-13582.php) |
| **SoC / Prosesor** | MediaTek Dimensity 8400-Ultra (4nm, Octa-core Cortex-A725 hingga 3.25 GHz) |
| **GPU** | Arm Mali-G720 MC7 |
| **Driver Vulkan yang Digunakan** | `Driver Sistem Vulkan (Universal - Kompatibel Semua GPU)` |
| **Skala Resolusi** | `1.0x (720p) - Standar Native Switch (Rekomendasi)` |
| **Frame Rate (FPS)** | **30 FPS Stabil** (100% kecepatan asli game pada Overworld, Cutscene, dan Gym Battle) |
| **Status Audio & Musik** | Normal & sinkron (melalui backend `SDL3` + dekoder `FFmpeg` native) |
| **Kontrol Layar Sentuh** | Responsif (mendukung sembunyikan tombol, kustomisasi posisi/ukuran, & haptik) |

## ✨ Fitur Utama Versi 

* **Antarmuka Bahasa Indonesia Lengkap**: Seluruh menu peluncur (*HomeScreen*), dialog status ekstraksi data game, pengaturan kontroler, hingga pesan diagnostik memori telah diterjemahkan ke dalam Bahasa Indonesia.
* **Ekstraktor NSP/XCI Mandiri di Perangkat (`liblgimport.so`)**: Mengimpor dan mendekripsi file `.nsp`, `.xci`, atau `.zip` secara langsung di perangkat Android menggunakan `prod.keys` milik pengguna ke dalam direktori `extracted/exefs/main` dan `extracted/romfs.bin`.
* **Dukungan Ganda Driver GPU Vulkan**:
  * **Driver Sistem Vulkan (Universal)**: Kompatibel untuk semua GPU (termasuk **Mali-G720** pada POCO X7 Pro, Immortalis, dan Xclipse).
  * **Driver Kustom Turnip / Adreno**: Menyertakan paket driver bawaan `Mesa Turnip v26.3.0-20261015-r5-8G2-OneUI` (`Vulkan 1.4.363` oleh *The412Banner*) di dalam `assets/turnip-8g2.zip` serta fitur impor driver `.zip` eksternal untuk pengguna Snapdragon.
* **Opsi Grafis & Skala Resolusi Fleksibel**:
  * Pilihan skala resolusi: `0.5x (480p) - Performa Tinggi`, `0.75x (540p) - Seimbang`, `1.0x (720p) - Standar Native Switch (Rekomendasi)`, dan `1.5x (1080p) - Definisi Tinggi (Tajam)`.
  * Opsi **Peredam Bergerigi (SMAA)** untuk memperhalus tepi objek 3D.
* **Kontrol Layar Sentuh (Joy-Con Eden Overlay)**:
  * Mode **Atur Tata Letak** interaktif: geser tombol untuk memindah posisi, cubit dua jari (*pinch-to-zoom*) untuk mengubah ukuran, atau gunakan *slider* ukuran & jarak tombol D-pad/ABXY.
  * Pengaturan **Transparansi Kontrol Sentuh** (`30% Transparan`, `50% Halus`, `70% Standar Rekomendasi`, `90% Jelas`, hingga `100% Solid / Pekat`) serta tombol cepat untuk menyembunyikan overlay di dalam game.
  * Dukungan **Giroskop (Aim)** menggunakan sensor rotasi perangkat, **Getaran (Haptik)**, serta kompatibilitas penuh kontroler eksternal via Bluetooth/USB (Xbox, PlayStation, Joy-Con, dan Steam Controller).
* **Pilihan 9 Bahasa Di Dalam Game**: Mendukung Bahasa Inggris, Jepang, Prancis, Jerman, Italia, Spanyol, Korea, Mandarin Sederhana, dan Mandarin Tradisional.
* **Pencatat Log Terintegrasi (`LogActivity`)**: Memudahkan pelacakan *error* dengan fitur lihat, bagikan, dan ekspor langsung ke `Downloads/letsgo-log.txt`.

## 📦 Bedah Isi & Arsitektur Teknis APK

Tabel berikut merinci struktur internal dari file 

| Komponen / Path di Dalam APK | Ukuran | Fungsi & Deskripsi Teknis |
| :--- | :--- | :--- |
| `lib/arm64-v8a/liblasgame.so` | `141.00 MB` | Binary utama kode game *Pokémon: Let's Go, Pikachu!* hasil rekompilasi statis AArch64 (`0x2000000000`–`0x2320000000`). |
| `lib/arm64-v8a/liblasrecomp.so` | `8.18 MB` | Mesin runtime `LAS Recomp`, penerjemah panggilan sistem (HLE), manajemen shader SPIR-V, dan perender Vulkan 1.3 + ImGui. |
| `lib/arm64-v8a/liblasreserve.so` | `7.96 KB` | Modul reservasi ruang alamat virtual memori agar rentang alamat tetap game tidak ditempati oleh proses sistem Android. |
| `lib/arm64-v8a/liblgimport.so` | `251.53 KB` | Pustaka native C/C++ untuk memverifikasi `prod.keys`, mendekripsi NCA, serta mengekstrak `ExeFS` dan membangun `romfs.bin`. |
| `lib/arm64-v8a/libSDL3.so` | `2.38 MB` | Pustaka SDL3 untuk manajemen jendela layar penuh, audio rendah latensi, sensor giroskop, dan input HID/Bluetooth. |
| `lib/arm64-v8a/libavcodec.so` dkk. | `12.97 MB` | Rangkaian pustaka FFmpeg (`avcodec`, `avformat`, `avutil`, `swscale`, `swresample`) untuk dekode video & audio in-game. |
| `lib/arm64-v8a/libc++_shared.so` | `6.77 MB` | Runtime standar C++ GNU/LLVM untuk Android NDK (`arm64-v8a`). |
| `assets/turnip-8g2.zip` | `2.64 MB` | Paket driver `libvulkan_freedreno.so` (Mesa Turnip v26.3.0 r5 8G2 OneUI, Vulkan 1.4.363) untuk perangkat ber-GPU Adreno. |
| `assets/fonts/DejaVuSans.ttf` | `759.72 KB` | Font bawaan untuk perenderan teks UI sistem dan layar diagnostik. |
| `classes.dex` & `AndroidManifest.xml` | `175.61 KB` | Kode aplikasi Android (`com.letsgorecomp`), UI Peluncur Bahasa Indonesia, *Joy-Con Eden Overlay*, dan pengelola izin penyimpanan. |

## 🚀 Panduan Instalasi & Persiapan Pertama Kali

1. **Siapkan File Game & Kunci**:
   * Buat sebuah folder di penyimpanan internal ponsel Anda (misalnya `/storage/emulated/0/LetsGo/`).
   * Masukkan file dump game **Pokémon: Let's Go, Pikachu! (v1.0.0)** berformat `.nsp` (tidak terkompresi/bukan `.ncz`) serta file `prod.keys` (dan `title.keys` bila diperlukan) ke dalam folder tersebut.
2. **Pasang & Buka Aplikasi**:
   * Instal file `LetsGoRecomp.apk` pada perangkat Android 9.0+ (ARM64).
   * Berikan izin akses penyimpanan saat diminta.
3. **Impor Data Game**:
   * Ketuk tombol **Impor Game (NSP)** lalu arahkan ke folder tempat Anda menyimpan file `.nsp` dan `prod.keys` (atau pilih **Gunakan Folder Hasil Ekstrak** jika Anda sudah memiliki folder `extracted/exefs/main` dan `extracted/romfs.bin`).
   * Tunggu hingga proses ekstraksi selesai dan muncul status hijau: *"Game Siap Dimainkan! Data game dan kunci berhasil diverifikasi."*
4. **Konfigurasi Driver GPU (Penting untuk POCO X7 Pro / GPU Mali)**:
   * Bagi pengguna **POCO X7 Pro** (Dimensity 8400-Ultra / Mali-G720) atau chipset MediaTek/Exynos lainnya, pastikan opsi **Driver GPU** diatur ke **Driver Sistem Vulkan (Universal - Kompatibel Semua GPU)**.
   * Bagi pengguna Snapdragon (Adreno), Anda dapat memilih driver Turnip bawaan atau mengimpor file `.zip` Turnip yang sesuai dengan seri Adreno Anda.
5. **Mulai Bermain**:
   * Pilih **Skala Resolusi** (`1.0x (720p)` direkomendasikan untuk keseimbangan ketajaman dan suhu perangkat).
   * Ketuk tombol **MAINKAN POKEMON LET'S GO PIKACHU**.

## 🛠️ Pemecahan Masalah Umum (Troubleshooting)

* **Pesan Error Ruang Alamat Memori (`0x2000000000 - 0x2320000000`)**:
  * *Penyebab*: Kode hasil rekompilasi statis membutuhkan alamat memori tetap `0x2000000000`–`0x2320000000`. Pada beberapa perangkat atau ROM Android tertentu, sebagian rentang alamat tersebut sudah dipakai oleh sistem sebelum aplikasi memintanya.
  * *Solusi*: Tutup paksa aplikasi (*Force Stop*) lalu buka kembali, atau periksa rincian rentang yang menghalangi melalui menu **Lihat Log** (`Downloads/letsgo-log.txt`).
* **Impor Berhenti atau `extracted/exefs/main` Tidak Terbentuk**:
  * Pastikan file `.nsp` adalah *base game* versi `1.0.0` yang utuh (bukan hanya file *Update* saja dan bukan format terkompresi `.ncz`), serta pastikan `header_key` di dalam `prod.keys` Anda valid.

## 🔐 Informasi File & Verifikasi Checksum

Gunakan *hash* berikut untuk memastikan file APK yang Anda unduh dari halaman *Releases* utuh dan tidak dimodifikasi:

* **Nama File**: `LetsGoRecomp-1.0.9-indonesia-54mb (2).apk`
* **Nama Paket**: `com.letsgorecomp`
* **Versi**: `1.0.9-indonesia`
* **Ukuran File**: `54.39 MB` (`57,030,385 bytes`)
* **MD5**: `e1058bf3d6f3f1bbe1882c2e5c7db201`
* **SHA-256**: `f6b94a74278853eef7d953f90f3b2ba9f6d051af38797fa322c8df616f5cf52e`

# 🎨 Alternatif Varian Markdown (Siap Salin untuk GitHub)

Jika Anda menginginkan tampilan yang lebih ringkas atau membutuhkan deskripsi khusus untuk halaman **GitHub Releases**, Anda dapat menyalin salah satu dari dua varian di bawah ini:

## Varian 2: Tampilan Minimalis & Ringkas (Compact README)

```markdown
# 🎮 Pokémon: Let's Go, Pikachu! — Android Recomp

Port Android berbasis **Static Recompilation (ARM64 + Vulkan 1.3)** dari proyek [letsgo-recomp](https://github.com/raulzrrs/letsgo-recomp) dengan antarmuka **Bahasa Indonesia** dan kontrol **Joy-Con Eden Overlay**.

### 📸 Tested on Xiaomi POCO X7 Pro (Dimensity 8400-Ultra / Mali-G720) — 30 FPS!
| Cutscene Pikachu (Overlay Aktif) | Overworld Gym (Overlay Sembunyi) | Gym Leader Brock Battle |
| :---: | :---: | :---: |
| ![Cutscene Pikachu](1003817023.jpg) | ![Overworld](1003817024.jpg) | ![Brock Battle](1003817022.jpg) |

### ⚡ Keunggulan Versi Ini
- **Bukan Emulator Biasa**: Kode game direkompilasi statis ke native ARM64 (`liblasgame.so`).
- **UI Bahasa Indonesia**: Menu peluncur, pengaturan resolusi (`480p`–`1080p`), SMAA, dan ekspor log sudah berbahasa Indonesia.
- **Kompatibel Semua GPU**: Berjalan mulus di GPU **Mali** (POCO X7 Pro) menggunakan *Driver Sistem Vulkan Universal* maupun **Adreno** (dilengkapi *Mesa Turnip v26.3.0* bawaan).
- **Joy-Con Eden Overlay**: Ukuran, posisi, dan transparansi tombol sentuh dapat diatur bebas + dukungan Giroskop & Haptik.

### 📥 Cara Pakai Singkat
1. Instal `LetsGoRecomp.apk.
2. Siapkan file `.nsp` (*Pokémon: Let's Go, Pikachu! v1.0.0*) dan `prod.keys` di satu folder.
3. Pilih **Impor Game (NSP)** di aplikasi dan tunggu verifikasi selesai.
4. Untuk pengguna **POCO X7 Pro / GPU Mali**, pilih **Driver Sistem Vulkan (Universal)** lalu ketuk **MAINKAN**.
```

## Varian 3: Template Catatan Rilis (GitHub Release Notes)

```markdown
## 🚀 Rilis LetsGoRecomp

Rilis ini menghadirkan port rekompilasi statis native ARM64 (`com.letsgorecomp`) untuk Android dengan terjemahan antarmuka penuh ke dalam **Bahasa Indonesia**.

### 📋 Apa yang Baru & Fitur Paket
* **Lokalisasi Bahasa Indonesia**: Seluruh menu utama, pengimpor NSP/XCI, pengaturan grafis, dan editor tata letak tombol kini dalam Bahasa Indonesia.
* **Tested on POCO X7 Pro (Mali-G720 MC7)**: Teruji berjalan stabil pada **30 FPS** menggunakan *Driver Sistem Vulkan (Universal)* pada resolusi Native 720p (`1.0x`).
* **Bundled Turnip Driver (Adreno)**: Menyertakan `Mesa Turnip v26.3.0-20261015-r5-8G2-OneUI` (`Vulkan 1.4.363`) di dalam APK untuk pengguna Snapdragon.
* **Joy-Con Eden Overlay**: Mendukung mode edit tata letak (*drag & pinch-to-zoom*), pengaturan transparansi (`30%`–`100%`), sembunyikan tombol, giroskop, dan getaran haptik.

### 🔒 Verifikasi File APK
* **File**: `LetsGoRecomp-1.0.9-indonesia-54mb (2).apk` (`54.39 MB`)
* **SHA-256**: `f6b94a74278853eef7d953f90f3b2ba9f6d051af38797fa322c8df616f5cf52e`
* **MD5**: `e1058bf3d6f3f1bbe1882c2e5c7db201`
```

## 🙏 Kredit & Apresiasi
* Proyek hulu rekompilasi statis: [raulzrrs/letsgo-recomp](https://github.com/raulzrrs/letsgo-recomp) (`LAS Recomp`).
* Driver Mesa Turnip Adreno bawaan: **The412Banner** (`Mesa Turnip v26.3.0-20261015-r5-8G2-OneUI`).
* Pustaka pendukung sumber terbuka: **SDL3**, **FFmpeg**, **Dear ImGui**, dan **DejaVu Fonts**.
