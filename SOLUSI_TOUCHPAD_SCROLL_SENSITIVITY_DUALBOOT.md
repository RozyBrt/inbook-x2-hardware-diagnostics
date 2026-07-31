# Solusi Penyesuaian Sensitivitas Scroll Touchpad Infinix INBook X2 (Fedora Linux)

Dokumen ini mencatat konfigurasi dan solusi teknis untuk mengatasi sensitivitas scroll 2-jari touchpad yang terlalu kencang (*hyper-sensitive*) pada laptop Infinix INBook X2 di sistem operasi Fedora Linux.

---

## 1. Root Cause & Analisis Driver

* **Fedora Linux (Wayland / libinput):** Secara default, driver `libinput` pada Fedora/GNOME (Wayland) tidak memiliki GUI bawaan untuk mengatur rasio scroll (*scroll factor*). Event dua jari dibaca dalam unit event piksel berpresisi tinggi yang menyebabkan efek scroll pada web browser dan GTK apps terasa sangat cepat meski jari hanya bergerak sedikit.

---

## 2. Implementasi Solusi (Fedora Linux)

### A. System-Wide Fix (`libinput-config`)
1. **Utility:** `libinput-config` (Repository: `https://gitlab.com/warningnonpotablewater/libinput-config.git`)
2. **Build & Install:**
   ```bash
   sudo dnf install -y gcc meson ninja-build libinput-devel
   cd ~/Documents/libinput-config
   meson build && cd build && ninja && sudo ninja install
   ```
3. **Konfigurasi System-Wide (`/etc/libinput.conf`):**
   ```ini
   scroll-factor=0.3
   ```

### B. Firefox Browser Level (Opsional / Tambahan)
1. Buka `about:config` di Firefox.
2. Cari `mousewheel.default.delta_multiplier_y`.
3. Ubah nilai default `100` menjadi `30` atau `40`.

---

## 3. Track Record & Chronological Execution Log

### [2026-07-31] - Iterasi 1: Penyesuaian Sensitivitas Touchpad Scroll (Fedora Linux)
* **Masalah:** Scroll 2 jari pada touchpad laptop Infinix INBook X2 bergerak sangat kencang di Fedora Linux meski pergerakan jari minimal.
* **Tindakan:**
  1. Melakukan clone repository `libinput-config` ke `~/Documents/libinput-config`.
  2. Mengonfigurasi `/etc/libinput.conf` dengan `scroll-factor=0.3`.
  3. Pengguna melakukan kompilasi, instalasi, dan restart sistem.
* **Status:** Selesai & Terverifikasi Aktif setelah reboot.
