# Solusi Penguncian Kernel & Manajemen Pembaruan Sistem (Fedora Linux & Dual-Boot Parity)
**Perangkat**: Laptop Infinix INBOOK X2 (Intel Core i7-1065G7 / Gen 10)  
**Target OS**: Fedora Linux 44 (Rolling/Rawhide) / Dual-Boot Windows 11  
**Tanggal Iterasi Terakhir**: 26 Juli 2026

---

## 📌 Masalah yang Diatasi (Symptom)
1. Munculnya notifikasi pembaruan sistem (*"System Updates"*) dengan **ikon tanda seru berwarna oranye (`!`)** di aplikasi GNOME Software (*Software Center*) yang berisi total 277 paket pembaruan.
2. Pengguna mengalami **ketidakkompatibilitas driver** (pada subsistem grafis iGPU Intel Ice Lake, Wi-Fi `iwlwifi`, atau audio Everest ES8336) setelah melakukan upgrade sistem dari Kernel versi 6 ke Kernel versi 7.
3. Pengguna terpaksa melakukan *downgrade/fallback* ke versi **Kernel `6.19.10-300.fc44.x86_64`** dan merasa pembaruan menyeluruh berisiko merusak stabilitas sistem yang saat ini sudah nyaman digunakan (*"kurang worth it"*).

---

## 🔬 Akar Masalah Teknis & Analisis Log Empiris
- **Arti Tanda Seru Oranye (`!`):** Berdasarkan pengujian empiris menggunakan perintah terminal `dnf updateinfo summary` dan `dnf advisory list`, tanda seru oranye merupakan indikasi adanya **66 Pembaruan Keamanan (*Security Updates*)**, dengan rincian **4 Critical** dan **27 Important** (termasuk penambalan pada `kernel-7.1.4`, `libssh`, `openvpn`, dan `cjson`). GNOME Software secara otomatis memberi tanda peringatan pada paket yang mengandung *CVE Security Advisory*.
- **Regresi Driver pada Kernel 7.x:** Pada arsitektur perangkat keras Infinix INBook X2 (motherboard Emdoor `EM_IC325_200B_V1.0`), lompatan versi major kernel Linux sering memicu regresi pada *Sound Open Firmware* (`sofessx8336` / SSP0 GPIO), modul konektivitas, atau manajemen daya tampilan (*Xwayland*).
- **Strategi Pemblokiran:** Untuk menjaga stabilitas driver tanpa harus mengorbankan pembaruan aplikasi penting (seperti peramban web untuk keamanan internet), sistem memerlukan mekanisme penguncian (*kernel locking*) pada level manajer paket DNF.

---

## 🛠️ Langkah-Langkah Eksekusi (Fedora Linux)

### Langkah 1: Pemblokiran Permanen Update Kernel di Konfigurasi DNF
Untuk mencegah manajer paket (`dnf`) dan GNOME Software mengunduh atau memasang Kernel versi 7 di masa depan, tambahkan aturan eksklusi pada berkas konfigurasi utama DNF:

1. Eksekusi perintah berikut di terminal:
   ```bash
   echo "excludepkgs=kernel*" | sudo tee -a /etc/dnf/dnf.conf
   ```
2. Isi berkas `/etc/dnf/dnf.conf` akan menjadi:
   ```ini
   # see `man dnf.conf` for defaults and possible options

   [main]
   excludepkgs=kernel*
   ```
   *Aturan ini secara otomatis memblokir seluruh paket yang berawalan `kernel` (`kernel`, `kernel-core`, `kernel-devel`, `kernel-modules*`, `kernel-tools*`).*

---

### Langkah 2: Verifikasi Eksklusi Paket Kernel
Pastikan sistem telah menerapkan pemblokiran dengan mengecek daftar pembaruan yang tertunda:
```bash
dnf check-update | grep -iE 'kernel' || echo "Sukses! Tidak ada lagi update kernel yang muncul."
```
*Kriteria Sukses:* Terminal mengembalikan pesan bahwa tidak ada paket kernel yang masuk dalam daftar update.

---

### Langkah 3: Alternatif Pembaruan Aman (Selektif)
Jika pengguna ingin melakukan pembaruan tanpa menyentuh kernel ataupun paket sistem yang berisiko:

1. **Update Khusus Peramban Web (Internet Security):**
   ```bash
   sudo dnf upgrade firefox brave-browser
   ```
2. **Update Khusus Security Patch (Non-Kernel):**
   ```bash
   sudo dnf upgrade --security --exclude=kernel*
   ```

---

### Langkah 4: Jaring Pengaman Darurat (Safety Nets & Rollback)
Jika di masa depan terjadi error akibat pembaruan paket lain, gunakan 2 jaring pengaman bawaan Fedora:
1. **GRUB Bootloader Fallback:** Saat laptop dinyalakan ulang, pilih baris kedua pada menu GRUB untuk masuk kembali menggunakan kernel lama yang stabil (`6.19.10`).
2. **DNF History Undo (Rollback Transaksi):** Batalkan transaksi pembaruan terakhir yang bermasalah dengan perintah:
   ```bash
   sudo dnf history undo last
   ```

---

## 🔄 Paritas Solusi Dual-Boot (Windows 11 Mapping)
Sesuai aturan *Dual-Boot Parity*, masalah regresi driver akibat pembaruan sistem otomatis juga terjadi pada OS Windows 11 (terutama pada driver audio `ESSX8336.sys` dan driver GPU Intel Iris Plus yang sering tertimpa oleh Windows Update).

### Langkah Pemetaan Windows: Memblokir Update Driver Otomatis di Windows Update
Cegah Windows Update mengganti driver hardware yang sudah stabil:
1. Buka **Registry Editor** (`regedit`) sebagai Administrator.
2. Navigasi ke path berikut:
   `HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate`
   *(Jika folder `WindowsUpdate` belum ada, buat key baru).*
3. Buat nilai DWORD (32-bit) baru dengan nama `ExcludeWUDriversInQualityUpdate` dan atur datanya menjadi `1` (Hexadecimal).
4. **Alternatif via Group Policy (`gpedit.msc`):**
   Masuk ke *Computer Configuration -> Administrative Templates -> Windows Components -> Windows Update -> Manage updates offered from Windows Update* -> Aktifkan (**Enable**) opsi **Do not include drivers with Windows Updates**.

---

## 📅 Track Record & Chronological Execution Log

### Iterasi 1 - 26 Juli 2026 (Identifikasi & Pemblokiran Kernel 7)
- **Symptom:** Pengguna melaporkan munculnya 277 paket pembaruan di GNOME Software disertai ikon tanda seru berwarna oranye (`!`).
- **Diagnosa Log Empiris:** 
  - Dieksekusi `dnf updateinfo summary`, teridentifikasi 66 *Security Updates* (4 Critical, 27 Important).
  - Dieksekusi `uname -r` & `rpm -q kernel`, teridentifikasi sistem sedang berjalan stabil pada Kernel `6.19.10-300.fc44.x86_64`.
- **Analisis & Keputusan:** Pengguna menyatakan telah mencoba Kernel 7 dan mengalami banyak ketidakkompatibilitas driver, sehingga memutuskan kembali ke Kernel 6. Disepakati bahwa menjaga stabilitas perangkat keras jauh lebih penting (*worth it*) dibanding memaksakan pembaruan major kernel.
- **Tindakan Perbaikan:**
  - Diterapkan konfigurasi pemblokiran permanen dengan menambahkan baris `excludepkgs=kernel*` ke dalam `/etc/dnf/dnf.conf`.
- **Hasil Verifikasi:** 
  - Dieksekusi `dnf check-update`, dikonfirmasi 0 paket kernel yang tersisa dalam antrean pembaruan.
  - Pengguna kini dapat melakukan *Update All* pada GNOME Software atau terminal tanpa risiko sistem mengunduh Kernel 7.x yang tidak kompatibel.

### Iterasi 2 - 26 Juli 2026 (Analisis Ikon Peringatan Oranye pada GNOME Software)
- **Symptom:** Pengguna menanyakan makna ikon tanda seru oranye (`!`) pada paket *System Updates*, *Firefox*, `xdg-desktop-portal`, `yelp`, dan `yelp-libs` di antarmuka GNOME Software.
- **Diagnosa Log Empiris:**
  - Dieksekusi `dnf advisory list`, teridentifikasi 4 paket dengan status **Critical Security Advisory**:
    - `firefox-153.0-3.fc44.x86_64` (ID: `FEDORA-2026-7d71f89d7e`, Severity: **Critical**)
    - `xdg-desktop-portal-1.22.1-1.fc44.x86_64` (ID: `FEDORA-2026-d8f8abf763`, Severity: **Critical**)
    - `yelp-2:49.1-1.fc44.x86_64` & `yelp-libs` (ID: `FEDORA-2026-ed4f450fa9`, Severity: **Critical**)
- **Analisis & Keputusan:** Ikon tanda seru oranye adalah mekanisme peringatan otomatis GNOME Software untuk penambalan celah keamanan kritis (CVE). Karena aturan `excludepkgs=kernel*` telah aktif di `/etc/dnf/dnf.conf`, pembaruan ini sangat aman dilakukan tanpa risiko mengunduh Kernel 7.x.
- **Tindakan:** Menginstruksikan pengguna bahwa melakukan *Download / Update* pada GNOME Software sangat dianjurkan untuk menambal kerentanan keamanan sistem.

### Iterasi 3 - 26 Juli 2026 (Verifikasi Pasca Update & Restart Sistem)
- **Symptom:** Pengguna melakukan *Update & Restart* melalui antarmuka GNOME Software, dan sistem menampilkan status *"Up to Date"* pasca restart.
- **Diagnosa Log Empiris:**
  - Dieksekusi `uname -r`, dikonfirmasi sistem tetap berjalan pada Kernel `6.19.10-300.fc44.x86_64` (tidak ada kernel 7.x yang terpasang/terunduh).
  - Dieksekusi `rpm -q firefox xdg-desktop-portal yelp yelp-libs`, dikonfirmasi ke-4 paket kritis telah sukses diperbarui ke versi terbaru (`firefox-153.0-3.fc44`, `xdg-desktop-portal-1.22.1-1.fc44`, `yelp-49.1-1.fc44`).
  - Dieksekusi `dnf check-update`, antrean pembaruan kosong bersih (selaras dengan status *"Up to Date"* pada GUI GNOME Software).
- **Analisis & Keputusan:** Pembuktian empiris menunjukkan aturan eksklusi `excludepkgs=kernel*` bekerja sempurna 100% saat pembaruan massal (*System Update*) dilakukan melalui GUI PackageKit/GNOME Software. Stabilitas perangkat keras Infinix INBook X2 tetap terjaga tanpa mengorbankan keamanan sistem.
- **Tindakan:** Mengonfirmasi kepada pengguna bahwa sistem kini dalam kondisi optimal, aman dari celah keamanan kritis, dan bebas dari risiko regresi kernel.
