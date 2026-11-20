# Driver WiFi RTL8189FS untuk Armbian ARM64

Repository ini berisi skrip dan patch untuk membangun serta memasang driver WiFi Realtek RTL8189FS pada Armbian ARM64. Pengujian utama dilakukan pada STB ZTE B860H V1 dengan SoC Amlogic S905X dan kernel `6.12.117-ophub`.

Driver menggunakan antarmuka SDIO dengan ID perangkat `024c:f179`. Skrip membantu memeriksa perangkat, menyiapkan source, menerapkan patch kompatibilitas `cfg80211`, membangun kernel module, memasang module, dan mengatur pemuatan otomatis saat boot.

## Perangkat dan kernel yang sudah diuji

| Komponen | Hasil pengujian |
|---|---|
| Perangkat | ZTE B860H V1 |
| SoC | Amlogic S905X |
| Sistem operasi | Armbian 26.11.0 Ubuntu Noble Server ARM64 |
| Kernel | `6.12.117-ophub` |
| Device tree | P212 |
| Chipset WiFi | Realtek RTL8189FS |
| ID SDIO | `024c:f179` |

Pada perangkat tersebut, driver berhasil dibangun menggunakan kernel header yang sesuai, dipasang, dan dimuat. Pemindaian jaringan WiFi berhasil dilakukan. NetworkManager juga berhasil terhubung ke access point, memperoleh alamat IP, dan melewatkan pengujian koneksi ke router, IP publik, serta DNS.

Setelah perangkat di-reboot, module dimuat kembali secara otomatis dan koneksi WiFi dapat digunakan.

Driver menampilkan interface `wlan0` dan `wlan1`. Keduanya merupakan interface virtual dari radio SDIO yang sama, bukan dua perangkat WiFi fisik. Gunakan `wlan0` sebagai interface utama.

**Catatan kompatibilitas:** Hasil di atas hanya berlaku untuk perangkat dan kernel yang diuji. ZTE B860H V2 dan perangkat lain belum diuji. Keberhasilan kompilasi atau kesesuaian vermagic saja tidak menjamin driver akan berfungsi pada perangkat lain.

## Instalasi

Pastikan Git tersedia:

```bash
git --version
```

Jika belum tersedia, pasang sebagai `root`:

```bash
apt update && apt install -y git
```

Atau dari akun biasa:

```bash
sudo apt update && sudo apt install -y git
```

Unduh repository:

```bash
git clone https://github.com/gungustiawan-arch/rtl8189fs-armbian-driver.git
cd rtl8189fs-armbian-driver
```

Jalankan installer sebagai pengguna biasa, tanpa menambahkan `sudo`:

```bash
bash install.sh
```

Untuk menyimpan direktori kerja dan log di HDD eksternal:

```bash
BUILD_DIR=/media/devmon/HardDisk/rtl8189fs-build bash install.sh
```

Secara default, proses build menggunakan dua pekerjaan paralel. Jika RAM terbatas, gunakan `JOBS=1`.

Installer akan mencari module lokal yang sesuai, kemudian memeriksa GitHub Release untuk module yang cocok dengan kernel aktif. Jika tidak ada module prebuilt yang dapat digunakan, installer akan membangun driver dari source upstream pada commit yang sudah ditentukan. Perubahan sistem yang membutuhkan hak akses administrator akan meminta `sudo` dan konfirmasi.

## Yang dilakukan installer

- Memeriksa arsitektur ARM64 dan ID perangkat SDIO `024c:f179`.
- Memeriksa kecocokan kernel header dengan kernel yang sedang berjalan.
- Mengambil source driver dari commit upstream yang sudah ditentukan.
- Menerapkan patch callback `cfg80211` untuk kernel 6.12.
- Menjalankan proses build dengan parameter kompatibilitas yang disiapkan oleh proyek.
- Memeriksa hasil module, nilai vermagic, dan dependensi `cfg80211`.
- Memasang module ke direktori `/lib/modules/KERNEL/updates/`.
- Menjalankan `depmod` dan mengatur pemuatan module saat boot.
- Membuat cadangan module lama sebelum penggantian.
- Menyimpan log build di `BUILD_DIR`.

Installer tidak mengganti Armbian, mengubah U-Boot, mengatur kredensial WiFi, atau mengonfigurasi NetworkManager.

Setelah build dari source selesai, source tree dan file sementara compiler akan dihapus secara default. Module hasil build dan log tetap disimpan di `BUILD_DIR`. Gunakan `KEEP_SOURCE=1` untuk mempertahankan source tree saat proses instalasi.

## Persyaratan

- Armbian/Linux ARM64.
- Kernel header yang cocok di `/lib/modules/$(uname -r)/build`.
- Perangkat SDIO dengan ID `024c:f179`.
- Koneksi internet untuk mengambil source upstream.
- Paket `git`, `patch`, `make`, GCC, `bc`, dan `kmod`.
- Ruang penyimpanan yang cukup untuk source, object file, module, dan log.

Jika paket build belum tersedia, installer akan menawarkan pemasangannya melalui `apt`.

## Prebuilt module

Installer mendukung module lokal dengan pola:

```text
prebuilt/KERNEL/8189fs.ko
```

Installer juga dapat mengambil module dari GitHub Release. Release harus menyediakan module dan checksum SHA-256 dengan nama:

```text
8189fs-KERNEL.ko
8189fs-KERNEL.ko.sha256
```

Sebelum module dibagikan, module harus berasal dari perangkat dan kernel yang sudah diuji. Skrip `scripts/stage-prebuilt.sh` menyalin module yang sedang terpasang ke direktori prebuilt untuk kernel aktif.

**Status saat ini:** Belum ada GitHub Release atau prebuilt module resmi pada repository ini. Pengujian instalasi menyeluruh melalui `install.sh` dan alur prebuilt masih belum selesai. Jangan menganggap module untuk kernel atau perangkat lain sudah kompatibel.

Lihat [Kebijakan Prebuilt Module](docs/PREBUILT-MODULE.md).

## Diagnostik dan penghapusan

Jalankan pemeriksaan:

```bash
bash scripts/diagnose.sh
```

Untuk menghapus driver:

```bash
sudo bash scripts/uninstall.sh
```

Skrip diagnostik menampilkan informasi kernel dan header, perangkat SDIO, metadata module, interface jaringan, serta pesan driver dari kernel log.

Skrip uninstall meminta konfirmasi sebelum menghapus module dan konfigurasi autoload milik proyek ini. Jika tersedia, skrip juga akan memulihkan module cadangan.

Dokumentasi lain:

- [Panduan Instalasi](docs/INSTALLATION.md)
- [Pemecahan Masalah](docs/TROUBLESHOOTING.md)
- [Pengujian Installer](docs/ACCEPTANCE-TEST.md)
- [Kebijakan Prebuilt Module](docs/PREBUILT-MODULE.md)

## Source driver dan patch

Source driver diambil dari [gustiarto/rtl8189fs-armbian](https://github.com/gustiarto/rtl8189fs-armbian), pada commit:

```text
786a5816a3731d57aee9eaa97744b89c4b799417
```

Repository ini menyimpan patch kompatibilitas callback `cfg80211` untuk kernel 6.12:

`patches/kernel-6.12/0001-cfg80211-set-monitor-channel-signature.patch`

GitHub Actions digunakan untuk memeriksa sintaks shell, menjalankan ShellCheck, dan memeriksa apakah patch dapat diterapkan pada commit upstream tersebut.

README pada repository upstream menyebutkan lisensi GPL-2.0. Source upstream diambil secara terpisah dan tidak disalin ke repository ini. Periksa lisensi upstream sebelum mendistribusikan ulang source atau binary.

## Catatan penggunaan

Proyek ini tidak resmi dan digunakan tanpa jaminan. Sediakan koneksi Ethernet atau jalur pemulihan lain sebelum memasang driver. Jangan mengganti device tree tanpa memastikan kecocokannya karena DTB yang salah dapat menyebabkan perangkat gagal boot.

Periksa kompatibilitas perangkat dan tinjau perubahan sistem sebelum menjalankan installer.
