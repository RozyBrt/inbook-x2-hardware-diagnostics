# Panduan Eksekusi Fix Display Glitch (Windows)
**Perangkat**: Laptop Infinix INBOOK X2 (Intel Core i7-1065G7 / Gen 10)  
**Target OS**: Windows 10 / Windows 11

---

## 📌 Masalah yang Diatasi (Symptom)
Layar laptop tiba-tiba *glitch* (garis-garis visual/berkedip) atau *blank* saat:
1. Menarik (*drag-and-drop*) file foto/video ke *timeline* aplikasi CapCut.
2. Melakukan *scrolling* cepat di aplikasi Microsoft Office (Word/Excel).

---

## 🔬 Akar Masalah Teknis (Windows Context)
- **Konflik Power Domain Audio & iGPU**: Saat file ditarik di CapCut, aplikasi tidak hanya merender *overlay* visual via iGPU, tetapi juga menginisialisasi *Audio Stream / Preview Waveform* secara bersamaan.
- **Audio Idle Wakeup**: Fitur Power Save pada Audio Controller di Windows mematikan audio setelah idle beberapa detik. Begitu CapCut/Office aktif, driver audio terbangun secara mendadak. Transisi daya yang bersamaan dengan beban rendering iGPU menyebabkan penurunan tegangan (*voltage sag*) singkat dan desinkronisasi sinyal layar eDP (*Embedded DisplayPort*).

---

## 🛠️ Langkah-Langkah Eksekusi (Untuk AI / Teknisi Windows)

### Langkah 1: Mematikan Audio Power Management (Fitur Hemat Daya Audio)
Cegah Windows mematikan/menidurkan *subsystem* audio yang memicu lonjakan daya saat terbangun:
1. Buka **Registry Editor** (`regedit`) sebagai Administrator.
2. Navigasi ke path berikut:
   `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Class\{4d36e96c-e325-11ce-bfc1-08002be10318}`
3. Cari folder sub-key (misalnya `0000`, `0001`, atau `0002`) yang mewakili perangkat audio Intel / Realtek.
4. Ubah atau buat nilai DWORD (32-bit) berikut:
   - `ConservationIdleTime` = `ffffffff` (Value Hexadecimal)
   - `IdlePowerState` = `00000000` (Value DWORD)
   - `PerformanceIdleTime` = `ffffffff` (Value Hexadecimal)

---

### Langkah 2: Mematikan Panel Self Refresh (PSR) pada GPU Intel
Cegah GPU mematikan sinyal layar saat transisi rendering:
1. Buka **Intel Graphics Command Center**.
2. Masuk ke **System** -> tab **Power**.
3. Matikan opsi **Panel Self Refresh (PSR)** (`Disabled`).
4. Matikan opsi **Display Power Saving Technology (DPST)** (`Disabled`).

---

### Langkah 3: Penyesuaian Akselerasi Hardware di CapCut
1. Buka aplikasi CapCut di Windows.
2. Masuk ke **Settings** -> **Performance**.
3. Pastikan opsi rendering menggunakan konfigurasi yang tidak bentrok dengan overlay DWM (gunakan opsi rendering Direct3D/DirectX 11 standar).

---

## 📋 Pengujian & Verifikasi (Windows)
1. Buka CapCut dan coba lakukan *drag & drop* berbagai file media ke *timeline* secara berulang.
2. Buka Microsoft Word / Excel dan lakukan *scrolling* cepat pada dokumen yang kaya tabel dan gambar.
3. **Kriteria Sukses**: Layar tetap stabil tanpa ada kedipan, garis *glitch*, atau layar hitam/blank.
