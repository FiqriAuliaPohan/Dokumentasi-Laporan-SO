# Sistem Manajemen Web Server dengan Fitur Monitoring dan Load Balancer

Proyek Tugas Akhir mata kuliah **Sistem Operasi**
Jurusan Teknik Informatika dan Komputer, Program Studi Teknik Informatika
**Politeknik Negeri Jakarta**

## Deskripsi Proyek

Proyek ini membangun sistem web server sederhana berbasis **Debian Linux** yang terdiri dari dua container **Nginx** (dijalankan dengan **Podman**), satu **HAProxy** sebagai *load balancer* dengan metode Round Robin, dan **Bash Shell Script** untuk memantau kondisi server serta ketersediaan layanan.

Permintaan dari pengguna diterima HAProxy terlebih dahulu, lalu dibagi secara bergantian ke dua backend Nginx agar beban tidak menumpuk di satu server. Selain itu, script monitoring mencatat penggunaan CPU, RAM, penyimpanan, serta status layanan ke dalam log/laporan sehingga gangguan bisa terdeteksi lebih awal.

Penjelasan lengkap (arsitektur, konsep Sistem Operasi, fitur, cara instalasi, dan pengujian) ada di [PENJELASAN.md](Penjelasan-tentang-projek.md).

## Tim (Kelompok 3)

| Nama | NIM |
|---|---|
| Ahmad Hannan Alvinsyah Syuhada | 2607411067 |
| Fiqri Aulia Pohan | 2607411087 |
| Guistino De Benjiro Hadinata | 2607411061 |
| Harya Wikanta Atmandashah Banowo | 2607411007 |
