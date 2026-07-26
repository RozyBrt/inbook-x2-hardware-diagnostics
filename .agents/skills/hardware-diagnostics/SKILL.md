---
name: hardware-diagnostics
description: Playbook for diagnosing hardware, kernel logs, power management, Bluetooth stack, and RF interference on Linux and Windows.
---

# Hardware Diagnostics Playbook

Skill ini digunakan untuk mendiagnosis masalah hardware pada laptop Infinix INBook X2 (Intel Core Gen 10 Ice Lake).

## Checklists Diagnosa Linux

1. **Bluetooth Controller & USB Power Control:**
   * Cek bus USB: `lsusb` dan `for dev in /sys/bus/usb/devices/*/power/control; do ...; done`
   * Periksa status power: `cat /sys/bus/usb/devices/<bus_id>/power/control`
   * Test aturan udev: `udevadm test /sys/bus/usb/devices/<bus_id>`
   * Peringatan Udev: Pastikan aturan buatan pengguna menggunakan awalan `99-` (misal `99-bluetooth-power.rules`) agar tidak tertimpa oleh `/usr/lib/udev/rules.d/60-autosuspend.rules` milik Fedora.

2. **Wi-Fi & Bluetooth 2.4 GHz Coexistence:**
   * Cek frekuensi Wi-Fi aktif: `iw wlo1 link`
   * Cek status Wi-Fi power save: `iw wlo1 get power_save`
   * Matikan Wi-Fi power save jika aktif: `iw dev wlo1 set power_save off`
   * Buat konfigurasi permanen NetworkManager: `/etc/NetworkManager/conf.d/disable-wifi-powersave.conf` (`wifi.powersave = 2`)

3. **BlueZ Daemon & Input Profile (HoG / GATT):**
   * Pantau log live: `journalctl -f -u bluetooth`
   * Cek pesan timeout: `profiles/input/device.c:hidp_report_req_timeout`
   * Setel `/etc/bluetooth/input.conf`: `UserspaceHID=true` dan `IdleTimeout=0` untuk menyerahkan penanganan ke kernel `uhid`.

## Checklists Diagnosa Windows

1. **Power Management Device Manager:**
   * `devmgmt.msc` -> Hilangkan centang "Allow the computer to turn off this device to save power" pada Bluetooth, HID LE Device, dan Wi-Fi Adapter.
2. **Registry USB Selective Suspend:**
   * Setel `DeviceSelectiveSuspended = 0` pada `HKLM:\SYSTEM\CurrentControlSet\Enum\USB\VID_8087&PID_0AAA`.
   * Setel `SystemRemoteWakeSupported = 1` dan `Hcibypass = 1` pada `BTHPORT\Parameters`.
