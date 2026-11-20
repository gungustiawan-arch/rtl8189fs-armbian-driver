# Pemecahan masalah

## Installer menyatakan perangkat SDIO tidak ditemukan

Skrip build mencari perangkat SDIO dengan vendor/device ID `024c:f179`. Periksa perangkat yang terdeteksi:

```bash
ls /sys/bus/sdio/devices/
```

Untuk melihat informasi perangkat, gunakan direktori SDIO yang muncul pada perintah sebelumnya, misalnya:

```bash
cat /sys/bus/sdio/devices/mmc0:0001:1/uevent
```

Jika perangkat tidak ditemukan, periksa modul WiFi, revisi board, inisialisasi SDIO, dan device tree yang sedang digunakan. Pengujian ZTE B860H V1 menggunakan device tree P212. Jangan mengganti DTB tanpa memastikan kecocokannya.

## Kernel header yang sesuai tidak ditemukan

Kernel header harus cocok dengan kernel yang sedang berjalan. Periksa:

```bash
uname -r
readlink -f /lib/modules/$(uname -r)/build
cat /lib/modules/$(uname -r)/build/include/generated/utsrelease.h
```

Pastikan nilai `UTS_RELEASE` pada header sama dengan keluaran `uname -r`. Pasang paket header Armbian/Ophub yang tepat untuk kernel aktif. Jangan membangun module menggunakan header generik atau header dari kernel lain.

## Patch kompatibilitas tidak dapat diterapkan

Skrip build menggunakan source upstream pada commit tetap:

```text
786a5816a3731d57aee9eaa97744b89c4b799417
```

Patch proyek disiapkan untuk revisi tersebut. Jika source checkout pernah diubah secara manual, simpan salinannya dan gunakan direktori `BUILD_DIR` baru untuk pengujian. Jangan memaksa patch diterapkan jika ada bagian yang ditolak.

## Kompilasi gagal

Skrip build saat ini menjalankan proses kompilasi dengan parameter:

```text
CONFIG_CC_HAS_MIN_FUNCTION_ALIGNMENT=
KCFLAGS=-Wno-error
```

Parameter tersebut diberikan oleh skrip build. Keduanya belum dibuktikan sebagai kebutuhan universal untuk semua kernel atau compiler.

Log lengkap disimpan dengan nama `build-KERNEL.log` di dalam `BUILD_DIR`. Periksa bagian akhir log untuk menemukan error kompilasi. Peringatan compiler tidak sama dengan build yang berhasil; skrip tetap memerlukan file `8189fs.ko` yang valid dengan `vermagic` sesuai kernel aktif.

Jika perintah build belum tersedia, jalankan kembali `bash install.sh` agar installer dapat memasang paket prasyarat. Jika kernel header tidak lengkap atau tidak tersedia, pasang header Armbian/Ophub yang benar terlebih dahulu.

## Module melaporkan simbol yang tidak dikenal

Periksa metadata dependensi dan pesan kernel:

```bash
modinfo 8189fs
sudo modprobe cfg80211
sudo modprobe 8189fs
dmesg | tail -n 80
```

Pastikan module dibangun menggunakan kernel header yang tepat untuk kernel aktif. Jangan memaksa memuat module yang dibuat untuk kernel lain.

## File module tersedia, tetapi WiFi tidak dapat terhubung

Kumpulkan laporan diagnostik:

```bash
bash scripts/diagnose.sh
```

Kemudian periksa:

```bash
rfkill list
ip link
nmcli device status
dmesg | grep -iE '8189|wlan|firmware|sdio' | tail -n 100
```

Pada pengujian ZTE B860H V1, `wlan0` digunakan sebagai interface utama. Interface `wlan0` dan `wlan1` merupakan interface virtual dari satu radio SDIO yang sama, bukan dua adapter fisik yang terpisah.

## Module tidak dimuat setelah kernel diperbarui

External module terikat pada kernel build tertentu. Setelah kernel diperbarui, pasang kernel header yang tepat untuk kernel baru lalu jalankan kembali proses build dan instalasi.

Jangan menganggap module dari kernel sebelumnya tetap kompatibel. Periksa `vermagic`, pastikan module dapat dimuat, lalu uji pemindaian WiFi, koneksi, konektivitas, dan pemuatan otomatis setelah reboot.

## Installer menampilkan peringatan kernel belum diuji

Saat ini skrip build masih menggunakan `6.12.114-ophub` sebagai versi kernel yang ditandai telah diuji secara khusus. Pengujian manual B860H V1 yang didokumentasikan di README menggunakan `6.12.117-ophub`.

Karena penanda versi di skrip belum diselaraskan, build pada `6.12.117-ophub` dapat menampilkan peringatan dan meminta konfirmasi. Catat peringatan tersebut; jangan menyamakan keberhasilan build otomatis dengan keberhasilan uji runtime.
