# Kebijakan prebuilt module

Prebuilt module bergantung pada kernel build tertentu. Dua kernel yang sama-sama berada pada keluarga versi 6.12 belum tentu dapat menggunakan module yang sama.

## Pemeriksaan sebelum menerbitkan prebuilt

Sebelum module dibagikan sebagai prebuilt, lakukan pemeriksaan berikut:

1. Bangun module menggunakan kernel header yang tepat untuk kernel target.
2. Pastikan bagian pertama dari keluaran `modinfo -F vermagic 8189fs.ko` sama persis dengan keluaran `uname -r`.
3. Pastikan `modinfo -F depends 8189fs.ko` mencantumkan `cfg80211`.
4. Pastikan module dapat dimuat dengan `modprobe 8189fs`.
5. Pastikan perangkat SDIO terdeteksi dan pemindaian jaringan WiFi berhasil.
6. Pastikan perangkat dapat terhubung ke jaringan dan memiliki konektivitas.
7. Pastikan module dimuat otomatis dan WiFi berfungsi setelah reboot.
8. Catat checksum SHA-256 serta detail perangkat, image Armbian, dan kernel yang diuji.

Kompilasi yang berhasil dan `vermagic` yang cocok belum cukup untuk menyatakan sebuah prebuilt siap digunakan. Pengujian saat driver dijalankan tetap diperlukan.

## Nama file pada GitHub Release

Untuk kernel dengan nama rilis `KERNEL`, GitHub Release harus menyediakan dua file berikut:

```text
8189fs-KERNEL.ko
8189fs-KERNEL.ko.sha256
```

Buat file checksum di direktori yang sama dengan module:

```bash
sha256sum 8189fs-KERNEL.ko > 8189fs-KERNEL.ko.sha256
```

Installer dirancang untuk mengambil kedua file dari GitHub Release, lalu memeriksa checksum, `vermagic`, dan dependensi `cfg80211` sebelum memasang module. Jika file yang sesuai tidak tersedia atau pemeriksaannya gagal, installer akan menggunakan jalur build dari source.

## Status repository

Saat ini:

- Belum ada prebuilt module resmi yang dipublikasikan di repository.
- Belum ada GitHub Release untuk driver ini.
- Module yang berhasil diuji pada B860H V1 berasal dari proses build dan instalasi manual.
- Pengujian menyeluruh melalui `install.sh` masih belum selesai.
- Alur pengambilan dan pemasangan prebuilt belum diuji.

Perangkat uji manual yang menjadi acuan:

| Komponen | Konfigurasi |
|---|---|
| Perangkat | ZTE B860H V1 |
| SoC | Amlogic S905X |
| Sistem operasi | Armbian 26.11.0 Ubuntu Noble Server ARM64 |
| Kernel | `6.12.117-ophub` |
| Device tree | P212 |
| ID SDIO | `024c:f179` |

Jangan menerbitkan binary sebagai prebuilt resmi sebelum seluruh pemeriksaan pada dokumen ini selesai dan hasil pengujiannya dicatat. Jangan menggunakan module dari kernel `6.1.170-ophub` atau `6.12.114-ophub` sebagai pengganti module untuk `6.12.117-ophub` tanpa build dan pengujian yang sesuai.

## Penamaan dan checksum

Nama direktori prebuilt lokal mengikuti rilis kernel secara persis:

```text
prebuilt/KERNEL/8189fs.ko
```

Simpan checksum bersama artefak rilis dan catat hasil verifikasi. Jangan mengubah nama file agar terlihat cocok dengan kernel lain; nama file tidak mengubah ABI atau kompatibilitas module.
