# Penjelasan Proyek

Dokumen ini menjelaskan tujuan, arsitektur, konsep Sistem Operasi yang diterapkan, fitur, cara instalasi, pengujian, dan monitoring pada proyek **Sistem Manajemen Web Server dengan Fitur Monitoring dan Load Balancer**.

Ringkasan proyek dan daftar anggota kelompok ada di [README.md](README.md).

## Tujuan

- Membangun sistem monitoring server sederhana menggunakan Bash Script.
- Menerapkan pengelolaan dan pemeriksaan status layanan menggunakan `systemd`.
- Menjalankan dan mengelola dua container Nginx yang terpisah dengan Podman.
- Mengimplementasikan HAProxy sebagai load balancer yang membagi request HTTP ke dua backend Nginx.
- Menghasilkan log atau laporan monitoring untuk mengetahui kondisi sistem dan layanan.

## Arsitektur

```
Client / Browser
       |
       |  HTTP Request
       v
 HAProxy (Load Balancer, port 80, Round Robin)
       |
       +-----------------------+
       |                       |
       v                       v
 Nginx 1 (Podman)        Nginx 2 (Podman)
  port host 8001          port host 8002
       |                       |
       v                       v
  Halaman Web 1           Halaman Web 2
```

**Alur singkat:**

1. Pengguna membuka alamat server lewat browser, lalu browser mengirim HTTP request.
2. Request diterima HAProxy terlebih dahulu, bukan langsung oleh web server.
3. HAProxy meneruskan request ke salah satu backend secara bergantian (Round Robin).
4. Nginx di dalam container memproses request dan mengembalikan halaman web ke pengguna.

## Teknologi yang Digunakan

| Komponen | Fungsi |
|---|---|
| Debian Linux (VM) | Sistem operasi utama |
| OpenSSH | Akses remote ke VM dari host (Windows) |
| Podman | Menjalankan container Nginx secara terisolasi |
| Nginx | Web server backend |
| HAProxy | Load balancer (Round Robin) dan halaman statistik |
| Bash Shell Script | Monitoring sumber daya dan layanan |

## Pemetaan Konsep Sistem Operasi

| Konsep SO | Penerapan di proyek |
|---|---|
| Process Management | Memantau proses HAProxy dan proses pendukung server dengan `ps` |
| Memory Management | Memantau penggunaan RAM dengan `free` |
| File System Management | Memantau kapasitas disk dengan `df` dan menyimpan hasil monitoring dalam file log/laporan |
| Shell Programming | Bash Script `container_monitor.sh` dan `service_monitor.sh` |
| Service Management | Memeriksa status layanan HAProxy dengan `systemd` |
| File Permission | Pengaturan izin eksekusi script dan hak akses direktori log/laporan |
| Process Isolation | Dua container Nginx terpisah yang dijalankan dengan Podman |

## Fitur

**Must have**
- Dua container Nginx berjalan di Podman
- HAProxy sebagai load balancer dengan metode Round Robin
- Halaman web berbeda di setiap backend
- Pemeriksaan status layanan HAProxy dan status container Podman
- Pemeriksaan respons HTTP tiap backend Nginx
- Hasil monitoring disimpan ke file log/laporan

**Should have**
- Waktu pemeriksaan pada setiap hasil monitoring
- Ringkasan hasil monitoring dalam format teks yang mudah dibaca

**Could have**
- Tampilan laporan monitoring yang lebih terstruktur
- Statistik HAProxy lewat halaman `/stats`
- Riwayat monitoring untuk membandingkan kondisi sistem dari waktu ke waktu

## Cara Instalasi

> Semua perintah dijalankan di VM Debian. Jalankan sebagai root atau gunakan `sudo`.

### 1. Instalasi dan konfigurasi SSH

Pastikan VM bisa ping ke internet dan ke komputer host. Cek IP VM dengan `ip a`.

```bash
apt update
apt install -y openssh-server
ssh -V
systemctl start ssh
systemctl status ssh
```

Edit konfigurasi SSH:

```bash
nano /etc/ssh/sshd_config
```

Pastikan (hapus tanda `#` pada) baris berikut:

```
Port 22
PermitRootLogin yes
PubkeyAuthentication yes
```

Terapkan konfigurasi, lalu uji dari Windows PowerShell:

```bash
systemctl restart ssh
```

```powershell
ssh username@IP-SERVER
```

> **Catatan keamanan:** `PermitRootLogin yes` dipakai agar pengerjaan lebih mudah di lingkungan lab/VM. Jangan dipakai di server produksi.

### 2. Instalasi dan konfigurasi Podman

```bash
apt install -y podman
podman --version
podman pull docker.io/library/nginx:latest
podman images
```

Buat folder untuk halaman web masing-masing container:

```bash
mkdir -p /data/web{1..2}
ls /data
```

Jalankan dua container Nginx:

```bash
podman run -d --name web1 -p 8001:80 -v /data/web1/:/usr/share/nginx/html nginx
podman run -d --name web2 -p 8002:80 -v /data/web2/:/usr/share/nginx/html nginx
```

Penjelasan opsi:

- `-d`: jalan di background (detached mode)
- `--name web1`: nama container
- `-p 8001:80`: port 8001 di host diarahkan ke port 80 di dalam container
- `-v /data/web1/:/usr/share/nginx/html`: folder host dihubungkan ke folder web root Nginx di container

Cek container, lalu buat halaman web untuk masing-masing:

```bash
podman ps
# jika belum jalan: podman start web1 web2

echo "Welcome to Web1 Coba Ngetes" > /data/web1/index.html
echo "Welcome to Web2 Coba Ngetes" > /data/web2/index.html
```

Uji lewat browser: `http://IP-VM:8001` dan `http://IP-VM:8002`.

### 3. Instalasi dan konfigurasi HAProxy

```bash
apt install -y haproxy
haproxy -v
nano /etc/haproxy/haproxy.cfg
```

Tambahkan konfigurasi berikut di bawah bagian `defaults`:

```
frontend web
        bind *:80
        default_backend app
        stats enable
        stats uri /stats

backend app
        balance roundrobin
        server web1 127.0.0.1:8001 check
        server web2 127.0.0.1:8002 check
```

Terapkan konfigurasi:

```bash
systemctl restart haproxy
```

**Penjelasan konfigurasi:**

- **frontend web**: pintu masuk request dari client.
  - `bind *:80`: menerima koneksi di semua alamat IP lewat port 80 (HTTP).
  - `default_backend app`: meneruskan request ke backend bernama `app`.
- **stats**: halaman statistik HAProxy.
  - `stats enable`: mengaktifkan fitur statistik.
  - `stats uri /stats`: halaman statistik bisa dibuka di `http://IP-VM/stats`.
- **backend app**: daftar server tujuan.
  - `balance roundrobin`: request dibagi bergantian ke server yang tersedia.
  - `server web1 127.0.0.1:8001 check`: mendaftarkan web1 dengan health check.
  - `server web2 127.0.0.1:8002 check`: mendaftarkan web2 dengan health check.

## Pengujian

1. Buka `http://IP-VM` di browser.
2. Refresh berkali-kali. Tampilan akan bergantian antara **Web1** dan **Web2**, tanda load balancing Round Robin berjalan.
3. Buka `http://IP-VM/stats` untuk melihat status frontend, backend, dan server (web1 dan web2 harus berstatus UP).

## Monitoring

Script monitoring ditulis dengan Bash dan memeriksa:

- Penggunaan CPU, RAM, dan kapasitas penyimpanan
- Status layanan HAProxy (`systemd`)
- Status container Podman (web1 dan web2)
- Respons HTTP dari masing-masing backend Nginx

Hasilnya dicatat beserta waktu pemeriksaan ke dalam file log/laporan.

```
.
├── container_monitor.sh   # pemeriksaan status container dan respons HTTP backend
├── service_monitor.sh     # pemeriksaan status layanan dan sumber daya sistem
└── logs/                  # hasil monitoring
```

Cara menjalankan:

```bash
chmod +x container_monitor.sh service_monitor.sh
./container_monitor.sh
./service_monitor.sh
```

> Sesuaikan nama file, folder, dan cara menjalankan dengan isi repo kalian.
