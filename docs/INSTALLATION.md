# Panduan instalasi dan build

## Perangkat yang sudah diuji

Pengujian manual yang berhasil dilakukan pada konfigurasi berikut:

| Komponen | Konfigurasi |
|---|---|
| Perangkat | ZTE B860H V1 |
| SoC | Amlogic S905X |
| Sistem operasi | Armbian 26.11.0 Ubuntu Noble Server ARM64 |
| Kernel | `6.12.117-ophub` |
| Device tree | P212 |
| WiFi | Realtek RTL8189FS |
| ID SDIO | `024c:f179` |

Hasil tersebut membuktikan driver dapat dibangun, dipasang, dimuat, digunakan untuk memindai dan terhubung ke jaringan WiFi, serta dimuat kembali setelah reboot pada perangkat uji tersebut.

**Batas pengujian:** hasil di atas berasal dari proses build dan instalasi manual. Pengujian installer otomatis dari awal sampai akhir masih belum selesai. Keberhasilan build pada kernel lain tidak otomatis membuktikan kompatibilitas saat dijalankan.

## 1. Siapkan repository

Pastikan Git tersedia:

```bash
git --version
```

Jika belum tersedia, pasang Git melalui pengelola paket Armbian/Debian, kemudian clone repository:

```bash
git clone https://github.com/gungustiawan-arch/rtl8189fs-armbian-driver.git
cd rtl8189fs-armbian-driver
```

## 2. Tentukan direktori build

Source, file hasil kompilasi, dan log membutuhkan ruang penyimpanan. Jika tersedia HDD eksternal, arahkan `BUILD_DIR` ke direktori pada disk tersebut:

```bash
BUILD_DIR=/lokasi/disk/rtl8189fs-build bash install.sh
```

Jika tidak ditentukan, direktori kerja default adalah:

```text
~/.local/share/hg680p-rtl8189fs
```

Installer menggunakan dua pekerjaan paralel secara default. Pada perangkat dengan RAM terbatas, gunakan satu pekerjaan:

```bash
JOBS=1 bash install.sh
```

## 3. Jalankan installer

Jalankan sebagai pengguna biasa, tanpa `sudo`:

```bash
bash install.sh
```

Installer meminta autentikasi `sudo` di awal dan menggunakan hak administrator untuk memasang paket yang diperlukan serta melakukan perubahan pada sistem. Jika paket build belum tersedia, installer dapat menjalankan `apt-get update` dan memasang paket seperti `build-essential`, `git`, `patch`, `bc`, atau `kmod`.

Kernel header harus cocok dengan kernel yang sedang berjalan. Jika paket `linux-headers-KERNEL` tidak tersedia, pasang header Armbian/Ophub yang tepat secara manual, lalu jalankan kembali installer.

## 4. Alur kerja installer

Secara berurutan, installer melakukan hal berikut:

1. Memastikan installer dijalankan oleh pengguna biasa.
2. Memeriksa ketersediaan utilitas yang diperlukan.
3. Mencari module lokal atau cache yang cocok dengan kernel aktif.
4. Mencoba mengambil prebuilt module dari GitHub Release jika tersedia.
5. Jika prebuilt yang valid tidak tersedia, menyiapkan paket build dan kernel header.
6. Menjalankan skrip build untuk memeriksa arsitektur ARM64, header kernel, dan perangkat SDIO `024c:f179`.
7. Mengambil source dari repository upstream pada commit yang sudah dipatok.
8. Memeriksa dan menerapkan patch kompatibilitas callback `cfg80211`.
9. Membangun module, lalu memeriksa `vermagic` dan dependensi `cfg80211`.
10. Menampilkan lokasi module dan meminta persetujuan sebelum instalasi sistem.
11. Memasang module ke `/lib/modules/KERNEL/updates/`, menjalankan `depmod`, dan mengatur pemuatan saat boot.

Hasil build dan log disimpan di `BUILD_DIR`. Source tree dan file sementara compiler dihapus secara default setelah build berhasil. Gunakan `KEEP_SOURCE=1` jika source tree perlu dipertahankan untuk pemeriksaan.

### Peringatan kernel yang perlu diketahui

Skrip build saat ini masih menganggap `6.12.114-ophub` sebagai kernel yang diuji secara khusus. Perangkat B860H V1 kita menggunakan `6.12.117-ophub`, yang sudah berhasil diuji secara manual. Karena perbedaan ini, skrip akan menampilkan peringatan dan meminta konfirmasi saat build pada kernel tersebut. Jangan mengabaikan peringatan untuk kernel lain tanpa pengujian.

### Catatan pemasangan module

Jika module lama sudah ada di lokasi tujuan, skrip instalasi menyalinnya ke:

```text
/var/backups/hg680p-rtl8189fs
```

Konfigurasi autoload dibuat di:

```text
/etc/modules-load.d/hg680p-rtl8189fs.conf
```

Jika file konfigurasi tersebut sudah ada dan tidak berisi satu baris `8189fs` secara terpisah, installer menolak mengubahnya.

Saat ini skrip instalasi belum memiliki mekanisme rollback otomatis untuk setiap kegagalan setelah file module atau konfigurasi mulai ditulis. Jika instalasi gagal, simpan pesan error dan log sebelum melakukan tindakan pemulihan.

## 5. Prebuilt module

Prebuilt module harus dibuat untuk kernel dan build yang tepat. Nama file Release yang didukung installer adalah:

```text
8189fs-KERNEL.ko
8189fs-KERNEL.ko.sha256
```

Checksum dibuat dengan:

```bash
sha256sum 8189fs-KERNEL.ko > 8189fs-KERNEL.ko.sha256
```

**Status saat ini:** repository belum memiliki prebuilt module resmi atau GitHub Release yang dipublikasikan. Installer akan menggunakan jalur build dari source jika tidak menemukan prebuilt yang lolos pemeriksaan. Jangan menggunakan module dari kernel lain hanya karena nomor versi utamanya sama.

Lihat [Kebijakan Prebuilt Module](PREBUILT-MODULE.md).

## 6. Diagnostik dan penghapusan

Untuk mengumpulkan informasi sistem dan driver:

```bash
bash scripts/diagnose.sh
```

Untuk menghapus instalasi yang dicatat oleh installer:

```bash
sudo bash scripts/uninstall.sh
```

Skrip uninstall meminta konfirmasi, mencoba melepas module jika sedang aktif, menghapus file module milik instalasi ini, menghapus konfigurasi autoload jika konfigurasi tersebut dibuat oleh installer, dan memulihkan cadangan terbaru jika tersedia.

**Perhatian:** proses pemulihan cadangan belum memeriksa `vermagic` module cadangan dan tidak otomatis memuat ulang module yang dipulihkan. Periksa kecocokan kernel sebelum menggunakan module hasil pemulihan.

## Catatan keselamatan

- Sediakan koneksi Ethernet atau jalur pemulihan lain sebelum memasang driver.
- Jangan mengganti device tree tanpa memastikan kecocokannya. Device tree yang salah dapat menyebabkan perangkat gagal boot.
- Kecocokan `vermagic` merupakan pemeriksaan penting, tetapi bukan jaminan kompatibilitas saat driver dijalankan.
- Setelah memasang driver pada perangkat atau kernel lain, uji pemindaian WiFi, koneksi, konektivitas, dan pemuatan setelah reboot.
