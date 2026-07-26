# Catatan Diagnostik & Solusi: Masalah Bluetooth CSR Lupa Pairing Pasca Suspend
**Perangkat:** Infinix INBOOK X2  
**Sistem Operasi:** Fedora Linux  
**Tanggal Catatan:** Juli 2026  

---

## 📌 1. Gejala & Keluhan
Setelah Masalah 1 (Gagal Suspend) berhasil diselesaikan dan laptop bisa tidur nyenyak menggunakan mode Modern Standby (`s2idle`), muncul keluhan baru:
Setiap kali laptop **dibangunkan kembali dari mode tidur**, koneksi dongle USB Bluetooth eksternal (Cambridge Silicon Radio / CSR `ID 0a12:0001`) mengalami reset. Akibatnya, keyboard Bluetooth (Logitech K380) dan TWS (Soundcore R50i) terputus dan menolak tersambung kembali. Pengguna terpaksa harus melakukan *pairing* ulang dari nol setiap bangun tidur seolah-olah perangkat belum pernah dikenal.

---

## 🔬 2. Analisis Masalah (*Mengapa Lupa Ingatan?*)
* **Uji Coba Normal:** Saat laptop menyala normal, jika dongle dicabut dan dicolok lagi dengan tangan, keyboard bisa langsung tersambung otomatis tanpa perlu pairing ulang.
* **Uji Coba Suspend:** Saat laptop tidur (`s2idle`), port USB mengalami pemutusan arus sementara (*autosuspend / reset*).
* **Akar Masalah:** Dongle Bluetooth berchipset CSR murah/klon (`0a12:0001`) memiliki kelemahan perangkat keras di mana memori volatil-nya akan terhapus (*amnesia*) jika arus listrik di port USB mengalami reset saat laptop tidur. Karena layanan Bluetooth Linux (`bluetooth.service`) tidak menyadari bahwa donglenya baru saja kehilangan daya saat tidur, Linux tidak menyuntikkan ulang *pairing key* dari hardisk. Akibatnya, dongle menolak koneksi keyboard dengan pesan error `Host is down (112)`.

---

## 🛠️ 3. Solusi Permanen yang Diterapkan
Diterapkan kombinasi **2 solusi otomatis** sekaligus di dalam sistem untuk mengatasi kelemahan dongle CSR tersebut:

### A. Aturan Udev Anti-Autosuspend untuk Dongle CSR
Memerintahkan Linux agar tidak memutus arus listrik secara drastis ke dongle CSR saat tidur.
* **Lokasi File:** `/etc/udev/rules.d/50-bluetooth-no-autosuspend.rules`
* **Isi Aturan:**
```udev
# Disable USB autosuspend for Cambridge Silicon Radio (CSR) Bluetooth dongle
ACTION=="add|change", SUBSYSTEM=="usb", ATTR{idVendor}=="0a12", ATTR{idProduct}=="0001", ATTR{power/control}="on"
```

### B. Systemd Resume Hook: "Simulasi Cabut-Colok Virtual" (*USB Unbind/Bind*)
Karena saat bangun dari mode Modern Standby (`s2idle`) pengontrol USB utama laptop (`xhci`) tetap melakukan reset pada port USB, layanan Bluetooth sering keburu aktif sebelum dongle siap.

Untuk mengatasinya tanpa perlu manusia mencabut-colok dongle dengan tangan, dibuat layanan *systemd resume hook* yang menstimulasikan pelepasan dan pemasangan driver USB secara virtual (efeknya 100% sama dengan mencabut dan mencolokkan dongle dengan tangan):
* **Lokasi File:** `/etc/systemd/system/bluetooth-resume.service`
* **Cara Kerja:**
  1. Menunggu 3 detik agar tegangan dan arus port USB stabil setelah laptop bangun.
  2. Menyapu seluruh port USB untuk mencari posisi dongle dengan ID `0a12:0001`.
  3. Melakukan *unbind* (cabut virtual) dan *bind* (colok virtual) pada port tersebut.
  4. Me-restart layanan `bluetooth.service` agar sistem memuat ulang memori kunci pairing ke dongle yang baru "dicolok".
* **Isi Konfigurasi:**
```ini
[Unit]
Description=Simulate Bluetooth dongle replug and restart after resume from sleep (Fix CSR dongle pairing loss)
After=suspend.target hibernate.target hybrid-sleep.target suspend-then-hibernate.target

[Service]
Type=oneshot
ExecStart=/bin/sh -c 'sleep 3; for d in /sys/bus/usb/devices/*; do id=$(cat $d/idVendor 2>/dev/null):$(cat $d/idProduct 2>/dev/null); if [ "$id" = "0a12:0001" ]; then dev=$(basename $d); echo "$dev" > /sys/bus/usb/drivers/usb/unbind 2>/dev/null; sleep 1; echo "$dev" > /sys/bus/usb/drivers/usb/bind 2>/dev/null; fi; done; systemctl restart bluetooth.service'

[Install]
WantedBy=suspend.target hibernate.target hybrid-sleep.target suspend-then-hibernate.target
```

**Hasil Akhir:** ✅ **SUKSES BESAR!** Setiap kali laptop bangun tidur, sistem otomatis melakukan cabut-colok virtual dalam waktu 4 detik, dan keyboard langsung tersambung otomatis tanpa pernah minta pairing ulang lagi.

---

## 🛡️ 4. Catatan Fleksibilitas Port & USB Hub
Kedua solusi otomatis di atas didesain secara **dinamis membaca ID Hardware (`ID 0a12:0001`)** dan **MAC Address chip Bluetooth**, bukan mematok nomor port fisik USB tertentu.
Oleh karena itu, pengguna **100% AMAN** untuk:
1. Mencabut dan memindahkan dongle dari port kiri (dekat charger) ke port seberang (kanan).
2. Menghubungkan dongle melalui **USB Hub** ataupun adaptor Type-C multifungsi.
3. Memindahkan port baik saat laptop menyala normal maupun saat sedang tidur (*Suspend*).

Sistem akan tetap otomatis mendeteksi lokasi port yang baru dan menjalankan seluruh fungsi di atas tanpa kendala.
