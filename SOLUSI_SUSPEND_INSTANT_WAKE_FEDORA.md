# Catatan Diagnostik & Solusi: Masalah Gagal Suspend (*Instant Wake*)
**Perangkat:** Infinix INBOOK X2  
**Sistem Operasi:** Fedora Linux  
**Tanggal Catatan:** Juli 2026  

---

## 📌 1. Gejala & Keluhan
Setiap kali pengguna mencoba menidurkan laptop dengan menekan tombol **Suspend** (Sleep) di menu atau menutup layar laptop, layar memang sempat mati sejenak namun dalam milidetik yang sama laptop langsung aktif/menyala kembali dengan sendirinya (*instant wake*). Laptop sama sekali tidak bisa tertidur lelap.

---

## 🔬 2. Riwayat Eksperimen (*Try & Error*) yang Gagal
Sebelum menemukan akar masalah aslinya, sempat dilakukan 3 kali percobaan untuk menebak komponen mana yang menginterupsi proses tidur dengan mematikan izin bangun (*ACPI Wakeup*) di `/proc/acpi/wakeup`:

| Eksperimen | Perangkat yang Dinonaktifkan | Alasan Hipotesis | Hasil | Mengapa Gagal? |
| :--- | :--- | :--- | :--- | :--- |
| **Percobaan 1** | `XHC` & `TXHC` (USB Controllers) | Mendeteksi adanya aktivitas reset pada port USB dongle dan koneksi keyboard saat laptop terbangun. | ❌ Gagal | Arus listrik dari port USB atau aktivitas mouse/keyboard bukan penyebab laptop batal tidur. |
| **Percobaan 2** | `RP09` (NVMe SSD) & `AWAC` (Alarm Timer) | SSD berjenis *DRAM-less* (Shenzhen Longsys SM2263XT) sering mengalami *bug* di Linux yang mengirim sinyal PCIe PME WAKE# palsu saat transisi daya. | ❌ Gagal | Masih langsung menyala kembali di detik yang sama. |
| **Percobaan 3** | `LID0` (Sensor Penutup Layar) | Mengetes kemungkinan sensor layar mengirim status "Layar Terbuka!" saat tombol suspend ditekan dengan posisi laptop terbuka. | ❌ Gagal | Proses tidur tetap dibatalkan secara instan oleh sistem. |

---

## 💡 3. Akar Masalah Sebenarnya
Melalui pemeriksaan log kernel (`journalctl`), ditemukan pesan error kunci:
> `ACPI: PM: Low-level resume complete`

Pesan ini muncul **di milidetik yang sama persis saat prosesor terakhir dipadamkan**, tanpa ada sinyal interupsi dari perangkat eksternal apa pun (USB/SSD/Layar).

* **Penyebab Utama:** Motherboard laptop modern (Intel Gen 10/11 ke atas) seperti Infinix INBOOK X2 didesain khusus oleh pabrikan untuk arsitektur **Windows Modern Standby (S0ix)**. Mereka sengaja menghapus atau merusak dukungan untuk protokol tidur lawas bernama **S3 (`deep`)** di dalam BIOS-nya.
* **Kesalahan Sistem:** Fedora Linux secara default mencoba memaksakan laptop tidur menggunakan protokol kuno `deep` (S3). Begitu perintah S3 dikirimkan, **BIOS langsung menolak perintah tersebut dan membuang error saat itu juga**. 
* **Kesimpulan:** Jadi laptop dari awal bukan terbangun karena diganggu mouse/bluetooth/SSD, melainkan **ditolak tidur oleh BIOS sejak milidetik pertama**!

---

## 🛠️ 4. Solusi Permanen yang Diterapkan
1. Semua setelan pemadaman sensor ACPI wakeup (`XHC`, `TXHC`, `RP09`, `AWAC`, `LID0`) dikembalikan ke setelan pabrik (`*enabled`).
2. Mode tidur Linux diganti secara permanen dari `deep` menjadi **`s2idle` (Modern Standby / S0ix)** melalui *bootloader* GRUB:
   ```bash
   sudo grubby --update-kernel=ALL --args="mem_sleep_default=s2idle"
   ```

**Hasil Akhir:** ✅ **SUKSES BESAR!** Laptop berhasil tertidur lelap mengikuti standar Modern Standby tanpa pernah mengalami *instant wake* lagi.
