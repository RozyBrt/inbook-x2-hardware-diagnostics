# 💻 Infinix INBook X2 System Diagnostic & Dual-Boot Workspace

Pusat dokumentasi, diagnostik sistem, dan *knowledge base* penyelesaian masalah hardware untuk laptop **Infinix INBook X2** yang beroperasi dalam sistem **Dual-Boot (Fedora Linux & OS Windows)**.

---

## 📌 1. Pemetaan Spesifikasi Perangkat Utama

| Komponen | Spesifikasi Hardware / Perangkat | Driver / Subsystem |
| :--- | :--- | :--- |
| **Model Laptop** | Infinix INBOOK X2 (Ice Lake Platform) | Motherboard Emdoor `EM_IC325_200B_V1.0` |
| **Prosesor (CPU)** | Intel Core i7-1065G7 (Gen 10) | Intel Iris Plus Graphics (iGPU) |
| **Sistem Operasi** | Dual-Boot: Fedora Linux 44 & Windows 11 | PipeWire (Linux) / WASAPI (Windows) |
| **Audio Codec** | Everest Semiconductor ES8336 | SOF (`sofessx8336`) / `ESSX8336.sys` (Windows) |
| **Wi-Fi & Bluetooth** | Intel Wireless-AC 9461 (CNVi 1x1 Combo) | `iwlwifi` & `btusb` / Intel Driver (Windows) |
| **Perangkat Bluetooth** | Logitech K380 Bluetooth LE Keyboard | BlueZ 5.86 HoG / Windows BTHPORT |
| **Perangkat Mouse** | Logitech M185 (2.4 GHz USB Dongle) | Port USB Kanan (Bebas Interferensi Charging) |

---

## 📂 2. Peta Struktur Direktori Workspace

```text
inbook-x2-hardware-diagnostics/
├── README.md                                    <-- [File Utama] Gambaran Umum & Indeks Workspace
├── PANDUAN_SOLUSI_AUDIO_MIC_LINUX_FEDORA.md       <-- [Dokumen] Solusi Paritas Audio Speaker & Digital Mic (✅ SOLVED)
├── SOLUSI_AUDIO_EVEREST_ES8336_FEDORA.md       <-- [Dokumen] Solusi Audio Speaker Everest ES8336 (SOLVED)
├── SOLUSI_BLUETOOTH_CSR_PAIRING_LOSS_FEDORA.md <-- [Dokumen] Solusi Bluetooth CSR Lupa Pairing Pasca Suspend
├── SOLUSI_BLUETOOTH_LOGITECH_K380_DUALBOOT.md  <-- [Dokumen] Solusi Koneksi Bluetooth K380 Dual-Boot (Linux & Windows)
├── SOLUSI_DISPLAY_GLITCH_WINDOWS.md            <-- [Dokumen] Solusi Display Glitch Windows (CapCut & Office)
├── SOLUSI_SUSPEND_INSTANT_WAKE_FEDORA.md       <-- [Dokumen] Solusi Masalah Instant Wake / Suspend di Fedora (✅ SOLVED)
├── SOLUSI_SYSTEM_UPDATE_KERNEL_LOCK_FEDORA.md  <-- [Dokumen] Solusi Penguncian Kernel & Manajemen Update (Fedora & Windows)
└── .agents/                                     <-- [Sistem AI] Otak, Aturan Main, & Playbook Keterampilan AI
    ├── AGENTS.md                                <-- Kitab Aturan Utama (SOP Diagnosa, No-Guessing, & Append-Only)
    └── skills/
        ├── dualboot-troubleshooting/
        │   └── SKILL.md                         <-- Playbook Pemetaan Solusi Linux ke Windows
        └── hardware-diagnostics/
            └── SKILL.md                         <-- Playbook Diagnosa Kernel, Udev Rules, & Bus Hardware
```

---

## 📄 3. Indeks Berkas Solusi Terintegrasi

Klik pada tautan berkas di bawah ini untuk mengakses panduan penyelesaian masalah secara langsung:

### 🔊 1. [SOLUSI_AUDIO_EVEREST_ES8336_FEDORA.md](./SOLUSI_AUDIO_EVEREST_ES8336_FEDORA.md)
* **Status:** ✅ **SOLVED (100% Verified)**
* **Masalah:** Speaker internal bisu/mati di Fedora Linux (Kernel >= 6.18+), volume mati mendadak di bawah 60%, dan kurva desibel ALSA melembek.
* **Solusi Utama:** Kompilasi driver kustom `es8336-fix` (penanganan inverted GPIO), penguncian parameter GRUB `snd_soc_sof_es8336.quirk=128`, dan aktivasi `api.alsa.soft-volume` pada WirePlumber.
* **Fitur:** Menyertakan *Definitive 5-Step Runbook* jika ingin mengaktifkan speaker di distro Linux lain di masa depan.

### 💤 2. [SOLUSI_SUSPEND_INSTANT_WAKE_FEDORA.md](./SOLUSI_SUSPEND_INSTANT_WAKE_FEDORA.md)
* **Status:** ✅ **SOLVED (Terverifikasi 29 Juli 2026)**
* **Masalah:** Laptop tidak bisa tidur (*instant wake*), langsung menyala kembali di milidetik yang sama saat tombol Suspend ditekan atau layar ditutup.
* **Akar Masalah:** Keyboard internal yang rusak secara fisik terus-menerus mengirim sinyal interupsi (IRQ 1) yang membangunkan laptop dari mode suspend.
* **Solusi:** Konektor keyboard internal dicabut secara fisik dari motherboard. Seluruh workaround software dibersihkan. Sistem berjalan bersih hanya dengan `mem_sleep_default=s2idle`.

### 🔗 3. [SOLUSI_BLUETOOTH_CSR_PAIRING_LOSS_FEDORA.md](./SOLUSI_BLUETOOTH_CSR_PAIRING_LOSS_FEDORA.md)
* **Masalah:** Dongle USB Bluetooth CSR (`0a12:0001`) lupa pairing setelah bangun tidur, menyebabkan keyboard Logitech K380 dan TWS terputus dengan error `Host is down (112)`.
* **Solusi Utama:** Aturan udev anti-autosuspend (`power/control="on"`) dan layanan `bluetooth-resume.service` untuk simulasi cabut-colok virtual (*unbind/bind*) secara otomatis setiap laptop bangun tidur.

### ⌨️ 4. [SOLUSI_BLUETOOTH_LOGITECH_K380_DUALBOOT.md](./SOLUSI_BLUETOOTH_LOGITECH_K380_DUALBOOT.md)
* **Masalah:** Koneksi keyboard Logitech K380 terputus-putus dan terjadi pengetikan karakter berulang (*stuck key*) di Linux & Windows.
* **Solusi Utama:** Aturan `99-bluetooth-power.rules` (USB power lock `on`), `UserspaceHID=true` di BlueZ, script PowerShell `apply-windows-fixes.ps1`, dan pemindahan dongle mouse 2.4 GHz ke port USB kanan.

### 🖥️ 5. [SOLUSI_DISPLAY_GLITCH_WINDOWS.md](./SOLUSI_DISPLAY_GLITCH_WINDOWS.md)
* **Masalah:** Layar laptop berkedip, garis visual glitch, atau blank saat *drag & drop* media di CapCut atau *scrolling* di Microsoft Word/Excel pada OS Windows.
* **Solusi Utama:** Nonaktifkan Audio Power Management di Registry Windows (`ConservationIdleTime = ffffffff`), matikan Panel Self Refresh (PSR) pada GPU Intel, dan setel akselerasi Direct3D di CapCut.

### 🛡️ 6. [SOLUSI_SYSTEM_UPDATE_KERNEL_LOCK_FEDORA.md](./SOLUSI_SYSTEM_UPDATE_KERNEL_LOCK_FEDORA.md)
* **Masalah:** Muncul 277 update dengan ikon peringatan oranye (`!`) di GNOME Software, dan ketidakkompatibilitas driver saat update dari Kernel 6 ke Kernel 7 yang menyebabkan regresi hardware.
* **Solusi Utama:** Pemblokiran permanen update kernel (`excludepkgs=kernel*`) di `/etc/dnf/dnf.conf`, panduan update selektif peramban/keamanan tanpa menyentuh kernel, dan pemblokiran update driver otomatis pada Windows Update.

### 🎙️ 7. [PANDUAN_SOLUSI_AUDIO_MIC_LINUX_FEDORA.md](./PANDUAN_SOLUSI_AUDIO_MIC_LINUX_FEDORA.md)
* **Status:** ✅ **SOLVED (100% Verified)**
* **Masalah:** Penyesuaian paritas spesifikasi audio Windows (`8086:34C8`, Subsystem `1A21:2782`) agar Speaker Internal & Digital Microphone (DMIC) dapat bekerja 100% bersamaan di Fedora Linux.
* **Solusi Utama:** Pengaktifan SOF DSP (`dsp_driver=3`), pengalokasian DMIC 2-channel (`dmic_num=2`), penguncian sample rate PipeWire di 48kHz (`99-clock-fix.conf`), aktivasi gain ALSA mixer (`Internal Mic` & `ADC PGA`), dan adapter pemetaan statis PipeWire Source (`98-internal-mic-source.conf`).

---


## ⚙️ 4. Konfigurasi Otak & Aturan AI (`.agents/`)

Ruang kerja ini dilengkapi dengan sistem konfigurasi agen otomatis di folder **[.agents/](./.agents/AGENTS.md)**:
* **[.agents/AGENTS.md](./.agents/AGENTS.md):** Berisi aturan bahwa AI wajib membaca log empiris (`dmesg`, `journalctl`, `regedit`), melarang penimpaan catatan lama (*append-only track record*), dan wajib menjaga keseragaman nama berkas.
* **[.agents/skills/](./.agents/skills/hardware-diagnostics/SKILL.md):** Berisi *playbook* panduan diagnosa kernel dan pemetaan solusi dual-boot.

---

## 🔄 5. Prosedur Penanganan Masalah Baru di Masa Depan

Jika di kemudian hari ditemukan masalah hardware/sistem baru di laptop ini:
1. Jalankan sesi diskusikan dengan AI di dalam ruang kerja ini.
2. AI akan membaca log empiris secara transparan tanpa menebak-nebak.
3. AI akan membuatkan berkas `.md` solusi baru (misal `SOLUSI_TOUCHPAD_INFINIX_X2.md`) tanpa merusak dokumen yang sudah ada.
4. Setiap iterasi penanganan akan dicatat secara kronologis (*append-only*).
