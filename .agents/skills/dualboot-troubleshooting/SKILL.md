---
name: dualboot-troubleshooting
description: Playbook for mapping Linux system fixes to Windows PowerShell scripts, Registry tweaks, and Device Manager settings.
---

# Dual-Boot Troubleshooting Playbook

Skill ini digunakan untuk mengonversi solusi teknis dari Linux ke OS Windows pada laptop dual-boot Infinix INBook X2.

## Matriks Ekivalensi Perbaikan (Linux vs Windows)

| Fitur / Komponen | Tindakan di Linux | Tindakan Ekuivalen di Windows |
| :--- | :--- | :--- |
| **USB Autosuspend** | `/etc/udev/rules.d/99-bluetooth-power.rules` + `btusb.enable_autosuspend=0` | `powercfg` USB selective suspend = Disabled + Registry `DeviceSelectiveSuspended=0` |
| **Wi-Fi Power Save** | `iw dev wlo1 set power_save off` + `/etc/modprobe.d/iwlwifi.conf` | Device Manager Wi-Fi `MIMO Power Save Mode = No SMPS` |
| **Audio Power Save** | `/etc/modprobe.d/infinix_audio_override.conf` (`power_save=0`) | Registry Audio Class `{4d36e96c-e325-11ce-bfc1-08002be10318}` (`ConservationIdleTime=ffffffff`) |
| **Display Panel Refresh** | GPU Power saving parameters / iGPU DRM power options | Intel Graphics Command Center: Matikan Panel Self Refresh (PSR) & DPST |

## Prosedur Pembuatan Script Windows (`apply-windows-fixes.ps1`)

Setiap kali perbaikan baru diidentifikasi di Linux, buatkan skrip PowerShell Administrator ekuivalen yang siap dijalankan saat pengguna booting ke Windows. Skrip harus menyertakan verifikasi nilai registry setelah perubahan diterapkan.
