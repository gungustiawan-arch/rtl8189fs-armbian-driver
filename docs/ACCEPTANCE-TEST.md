# Pengujian penerimaan installer

Dokumen ini digunakan untuk mencatat pengujian pertama installer secara menyeluruh pada ZTE B860H V1. Pengujian ini berbeda dari pengujian build dan instalasi manual yang sudah dilakukan sebelumnya.

## Persiapan

Gunakan direktori build baru agar hasil pengujian tidak bercampur dengan source dan direktori pengujian patch sebelumnya. Pilih lokasi penyimpanan yang memiliki ruang cukup, misalnya HDD eksternal:

```bash
BUILD_DIR=/lokasi/disk/rtl8189fs-installer-test bash install.sh
```

Jalankan sebagai pengguna biasa, bukan dengan `sudo`. Installer akan meminta autentikasi `sudo` saat diperlukan.

Saat ini belum ada prebuilt module yang diverifikasi dan dipublikasikan. Karena itu, installer seharusnya melanjutkan dengan mengambil source upstream pada commit yang sudah dipatok, lalu membangun module dari source.

**Catatan versi kernel:** skrip build saat ini masih menetapkan `6.12.114-ophub` sebagai versi kernel yang diuji secara khusus. Pada perangkat uji kita yang menggunakan `6.12.117-ophub`, skrip akan menampilkan peringatan dan meminta konfirmasi untuk melanjutkan. Peringatan ini belum diselaraskan dengan hasil pengujian manual B860H V1.

## Perilaku yang diharapkan

- Arsitektur perangkat terdeteksi sebagai ARM64.
- Kernel yang sedang berjalan dan kernel header yang sesuai berhasil diperiksa.
- Perangkat SDIO dengan ID `024c:f179` ditemukan.
- Patch kompatibilitas kernel 6.12 dapat diterapkan pada source upstream yang dipatok.
- Proses build selesai dan menghasilkan `output/8189fs.ko`.
- Nilai `vermagic` module sesuai dengan kernel yang sedang berjalan.
- Dependensi `cfg80211` tercantum pada metadata module.
- Installer menampilkan lokasi module dan meminta persetujuan sebelum memasang perubahan ke sistem.
- Module lama, jika ada, disalin ke direktori cadangan sebelum diganti.
- Konfigurasi pemuatan module saat boot dibuat jika belum tersedia.
- Installer menjalankan `depmod` dan mencoba memuat module jika belum aktif.

Jika salah satu tahap gagal, simpan log build di dalam `BUILD_DIR`. Jangan memaksa pemasangan module yang gagal melewati pemeriksaan.

## Pemeriksaan setelah instalasi

Pengujian manual sebelumnya pada B860H V1 sudah membuktikan pemindaian WiFi, koneksi ke access point, konektivitas jaringan, dan pemuatan module setelah reboot.

Pada pengujian installer ini, catat secara terpisah apakah alur otomatis build dan instalasi berhasil. Periksa kembali:

```bash
uname -r
modinfo 8189fs
lsmod | grep 8189fs
iw dev
nmcli device status
```

Jika module `8189fs` sudah aktif sebelum installer dijalankan, file module yang baru dipasang belum tentu langsung digunakan oleh module yang sedang berada di memori. Lakukan reload dengan aman atau reboot sebelum menyimpulkan bahwa versi baru sudah aktif.

## Status pengujian

- Build dan instalasi manual pada ZTE B860H V1: **berhasil**.
- Pemindaian dan koneksi WiFi setelah instalasi manual: **berhasil**.
- Pemuatan otomatis setelah reboot: **berhasil**.
- Pengujian menyeluruh melalui `install.sh`: **belum selesai**.
- Pengujian alur prebuilt module: **belum dilakukan**.

Jangan menerbitkan GitHub Release atau menyatakan installer sudah lolos pengujian sebelum seluruh alur otomatis selesai diuji dan hasilnya dicatat.
