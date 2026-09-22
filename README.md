# 🚀 Tutorial Deploy WordPress di AWS EC2 (LAMP Stack)

> Panduan lengkap dan santai untuk menginstal **WordPress** di atas **Amazon EC2**, mulai dari membuat *Key Pair*, konek pakai **PuTTY**, instalasi **LAMP** (Linux, Apache, MySQL, PHP), sampai troubleshooting kalau yang muncul malah halaman default Apache alih-alih situs WordPress kamu. 😄

---

## 📋 Daftar Isi

1. [Prasyarat](#-prasyarat)
2. [Membuat Key Pair di AWS](#-1-membuat-key-pair-di-aws)
3. [Membuat & Menjalankan Instance EC2](#-2-membuat--menjalankan-instance-ec2)
4. [Konversi Key Pair untuk PuTTY](#-3-konversi-key-pair-pem--ppk-untuk-putty)
5. [Konek ke EC2 via PuTTY](#-4-konek-ke-ec2-via-putty-menggunakan-public-ip)
6. [Instalasi LAMP Stack](#-5-instalasi-lamp-stack)
7. [Instalasi WordPress](#-6-instalasi-wordpress)
8. [Troubleshooting: Muncul Halaman Default Apache, Bukan WordPress](#-7-troubleshooting-yang-muncul-halaman-default-apache-bukan-wordpress)

---

## ✅ Prasyarat

Sebelum mulai, pastikan kamu sudah punya:

- [ ] Akun **AWS** (Free Tier cukup untuk latihan)
- [ ] **PuTTY** & **PuTTYgen** terinstal di komputer (untuk pengguna Windows) — [download di sini](https://www.putty.org/)
- [ ] Koneksi internet yang stabil
- [ ] Sedikit kesabaran ☕ (namanya juga belajar server)

---

## 🔑 1. Membuat Key Pair di AWS

Key Pair ini nantinya jadi "kunci rumah" buat masuk ke server EC2 kamu lewat SSH/PuTTY.

1. Login ke **AWS Management Console** → buka layanan **EC2**.
2. Di sidebar kiri, pilih **Key Pairs** (masih di bagian *Network & Security*).
3. Klik **Create key pair**.
4. Isi:
   - **Name**: bebas, misal `wordpress-key`
   - **Key pair type**: `RSA`
   - **Private key file format**:
     - Pilih **`.pem`** jika mau dikonversi manual ke PuTTY nanti
     - Atau langsung pilih **`.ppk`** kalau memang mau pakai PuTTY (lebih praktis!)
5. Klik **Create key pair** → file akan otomatis terdownload ke komputer kamu.

> ⚠️ **Penting:** Simpan file key pair ini baik-baik! Kalau hilang, kamu **tidak bisa** mendownload ulang dan berisiko tidak bisa masuk ke instance lagi.

---

## 🖥️ 2. Membuat & Menjalankan Instance EC2

1. Di dashboard EC2, klik **Launch Instance**.
2. Isi konfigurasi:
   - **Name**: misal `server-wordpress`
   - **AMI (Amazon Machine Image)**: pilih **Ubuntu Server 22.04 LTS** (atau versi LTS terbaru)
   - **Instance type**: `t2.micro` (Free Tier eligible)
   - **Key pair**: pilih key pair yang sudah dibuat di langkah sebelumnya
3. Pada **Network settings**, aktifkan/centang:
   - ✅ Allow SSH traffic (port 22)
   - ✅ Allow HTTP traffic (port 80)
   - ✅ Allow HTTPS traffic (port 443)
4. Klik **Launch Instance**.
5. Tunggu status instance berubah menjadi **Running**, lalu catat **Public IPv4 address**-nya — ini yang nanti dipakai untuk konek via PuTTY dan buka website.

---

## 🔄 3. Konversi Key Pair (.pem → .ppk) untuk PuTTY

PuTTY tidak bisa langsung membaca file `.pem`, jadi perlu dikonversi dulu pakai **PuTTYgen** (lewati langkah ini kalau kamu sudah punya file `.ppk` dari awal).

1. Buka aplikasi **PuTTYgen**.
2. Klik **Load**, ubah filter file ke *All Files (*.*)*, lalu pilih file `.pem` kamu.
3. Setelah berhasil dimuat, klik **Save private key**.
4. Akan muncul peringatan "tanpa passphrase" — klik **Yes** saja (untuk latihan, oke-oke saja).
5. Simpan file dengan ekstensi **`.ppk`**, misal `wordpress-key.ppk`.

---

## 🔌 4. Konek ke EC2 via PuTTY Menggunakan Public IP

1. Buka aplikasi **PuTTY**.
2. Di kolom **Host Name (or IP address)**, masukkan **Public IPv4 address** dari instance EC2 kamu.
3. Pastikan **Port** = `22` dan **Connection type** = `SSH`.
4. Di panel kiri, masuk ke **Connection → SSH → Auth → Credentials**.
5. Pada **Private key file for authentication**, klik **Browse** lalu pilih file `.ppk` hasil konversi tadi.
6. Kembali ke halaman **Session**, beri nama di **Saved Sessions** (misal `WordPress-EC2`) lalu klik **Save** supaya tidak perlu setting ulang tiap konek.
7. Klik **Open**.
8. Jika muncul *Security Alert*, klik **Accept**.
9. Saat diminta **login as**, ketik:
   ```
   ubuntu
   ```
   (gunakan `ec2-user` jika AMI yang dipakai adalah Amazon Linux)

🎉 Kalau berhasil, kamu akan masuk ke terminal server EC2 kamu!

---

## 🛠️ 5. Instalasi LAMP Stack

**LAMP** = **L**inux + **A**pache + **M**ySQL + **P**HP. Jalankan perintah berikut satu per satu di terminal PuTTY.

### a. Update sistem
```bash
sudo apt update && sudo apt upgrade -y
```

### b. Install Apache (Web Server)
```bash
sudo apt install apache2 -y
sudo systemctl enable apache2
sudo systemctl start apache2
```
Cek dengan membuka `http://<Public-IP-kamu>` di browser — kalau muncul halaman **"Apache2 Ubuntu Default Page"**, berarti Apache sudah jalan dengan baik ✅

### c. Install MySQL (Database Server)
```bash
sudo apt install mysql-server -y
sudo mysql_secure_installation
```
Ikuti wizard-nya: atur password root, hapus user anonim, disable remote root login, dsb (jawab `Y` untuk sebagian besar pertanyaan).

### d. Install PHP + modul pendukung WordPress
```bash
sudo apt install php libapache2-mod-php php-mysql php-curl php-gd php-mbstring php-xml php-xmlrpc php-zip php-intl -y
```

### e. Restart Apache
```bash
sudo systemctl restart apache2
```

---

## 🌐 6. Instalasi WordPress

### a. Download WordPress
```bash
cd /tmp
wget https://wordpress.org/latest.tar.gz
tar -xvzf latest.tar.gz
```

### b. Pindahkan file ke document root Apache
```bash
sudo cp -a /tmp/wordpress/. /var/www/html/
```

### c. Atur kepemilikan & permission
```bash
sudo chown -R www-data:www-data /var/www/html/
sudo find /var/www/html/ -type d -exec chmod 755 {} \;
sudo find /var/www/html/ -type f -exec chmod 644 {} \;
```

### d. Buat Database untuk WordPress
```bash
sudo mysql -u root -p
```
Lalu di dalam prompt MySQL:
```sql
CREATE DATABASE wordpress_db;
CREATE USER 'wp_user'@'localhost' IDENTIFIED BY 'PasswordKuat123!';
GRANT ALL PRIVILEGES ON wordpress_db.* TO 'wp_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### e. Konfigurasi `wp-config.php`
```bash
cd /var/www/html
sudo cp wp-config-sample.php wp-config.php
sudo nano wp-config.php
```
Isi bagian berikut sesuai database yang baru dibuat:
```php
define( 'DB_NAME', 'wordpress_db' );
define( 'DB_USER', 'wp_user' );
define( 'DB_PASSWORD', 'PasswordKuat123!' );
define( 'DB_HOST', 'localhost' );
```
Simpan dengan `CTRL + O` lalu `Enter`, keluar dengan `CTRL + X`.

### f. Buka di Browser 🎊
Akses `http://<Public-IP-kamu>` di browser, lalu ikuti wizard instalasi WordPress (isi judul situs, username admin, password, email).

---

## 🐞 7. Troubleshooting: Yang Muncul Halaman Default Apache, Bukan WordPress

Ini masalah **paling umum** setelah instalasi! Penyebabnya biasanya karena Apache masih membaca file `index.html` bawaan (default page), padahal WordPress memakai `index.php`. Berikut cara memperbaikinya:

### 🔍 Penyebab #1 — Masih ada file `index.html` bawaan Apache
File default `/var/www/html/index.html` diprioritaskan Apache dibanding `index.php`.

**Solusi:**
```bash
sudo rm /var/www/html/index.html
```
> Pastikan file `index.php` milik WordPress sudah ada di folder yang sama (`/var/www/html/`) sebelum menghapus file ini.

### 🔍 Penyebab #2 — Urutan `DirectoryIndex` belum memprioritaskan PHP
Apache secara default mencari `index.html` lebih dulu daripada `index.php`.

**Solusi:** edit file konfigurasi `dir.conf`
```bash
sudo nano /etc/apache2/mods-enabled/dir.conf
```
Ubah urutannya menjadi seperti ini (PHP paling depan):
```apache
<IfModule mod_dir.c>
    DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
```
Simpan lalu restart Apache:
```bash
sudo systemctl restart apache2
```

### 🔍 Penyebab #3 — Modul PHP belum aktif di Apache
Jika file PHP malah ter-*download* alih-alih dieksekusi, artinya `mod_php` belum aktif.

**Solusi:**
```bash
sudo a2enmod php8.1   # sesuaikan versi PHP yang terinstal
sudo systemctl restart apache2
```

### 🔍 Penyebab #4 — File WordPress salah lokasi / masih di dalam subfolder `wordpress/`
Kadang saat *copy-paste*, file WordPress ikut ter-copy beserta folder `wordpress/`-nya, sehingga struktur jadi `/var/www/html/wordpress/index.php`, bukan langsung di `/var/www/html/index.php`.

**Solusi:** pastikan isi folder `wordpress` (bukan foldernya) yang dipindah:
```bash
sudo cp -a /tmp/wordpress/. /var/www/html/
```
(perhatikan tanda titik `.` di akhir path sumber — itu kunci utamanya!)

### 🔍 Penyebab #5 — Cache browser
Kadang server sudah benar, tapi browser masih menyimpan cache halaman default lama.

**Solusi:** lakukan *hard refresh* (`CTRL + SHIFT + R`) atau buka lewat mode Incognito/Private.

### ✔️ Checklist Cepat Troubleshooting

| Cek | Perintah |
|---|---|
| File `index.html` sudah dihapus? | `ls /var/www/html/` |
| `index.php` WordPress ada di root? | `ls /var/www/html/wp-config.php` |
| `DirectoryIndex` sudah prioritaskan PHP? | `cat /etc/apache2/mods-enabled/dir.conf` |
| Modul PHP aktif? | `apache2ctl -M \| grep php` |
| Apache sudah di-restart? | `sudo systemctl status apache2` |

<p align="center">
  Dibuat dengan ❤️ untuk belajar deployment WordPress di AWS EC2
</p>
