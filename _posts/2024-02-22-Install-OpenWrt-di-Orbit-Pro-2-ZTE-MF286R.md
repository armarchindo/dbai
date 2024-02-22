---
title: Install OpenWrt di Orbit Pro 2 - ZTE MF286R
tags: [OpenWRT, Linux, QCA9563]
style: 
color: primary
description: Install OpenWrt di Orbit Pro 2 - ZTE MF286R
---

# OpenWrt Orbit Pro 2 - ZTE MF286R

# PERSIAPAN 

1. Reset Modem ke Pabrikan (Factory Reset)
2. Ubah IP DHCP Server Modem ("IP Address" dan "DHCP IP Pool"):
   - Default `192.168.8.1 -> 192.168.0.1`
3. Ubah IP Interface PC ke Static:
   - IP address: `192.168.0.22`
   - Subnet mask: `255.255.255.0`
   - Default gateway: `192.168.0.1`

# STEP 1: Mendapatkan Akses Root

- Dari 3 metode yang tertulis di dokumentasi blog [openwrt.org](https://openwrt.org) hanya 1 yang bisa yaitu via USB Serial TTL dengan baudrate speed `115200`
- Cek foto untuk mengetahui `TX RX GND`
  <p align="left">
    <img src="https://media.discordapp.net/attachments/1210095423201611827/1210176822663315496/image.png?ex=65e99bfe&is=65d726fe&hm=c3bf20065c4fd189b4b119b70fb80733d7abadebb7440944593cb475909d9c49&=&format=webp&quality=lossless&width=573&height=683" width="75%" />
  </p>
- Hidupkan router, ketika sudah benar-benar hidup, masuk ke terminal serial lalu tekan ENTER

# STEP 2: Backup Firmware Original

- Jalankan aplikasi `TFTPD Server` kemudian set "Server Interface" ke IP static yang kita set tadi yaitu `192.168.0.22`
- Input command `cat /proc/mtd` kemudian cocokkan dengan command backup partisi di bawah untuk recheck dan pastikan sama
- Copy paste 17 command backup partisi di bawah ini dan pastikan 17 partisi sudah terbackup semua sebelum lanjut ke step selanjutnya
```bash
tftp -l /dev/mtd0ro -r mtd0ro_uboot.img -p 192.168.0.22
tftp -l /dev/mtd1ro -r mtd1ro_uboot-env.img -p 192.168.0.22
tftp -l /dev/mtd2ro -r mtd2ro_reserved1.img -p 192.168.0.22
tftp -l /dev/mtd3ro -r mtd3ro_fota-flag.img -p 192.168.0.22
tftp -l /dev/mtd4ro -r mtd4ro_art.img -p 192.168.0.22
tftp -l /dev/mtd5ro -r mtd5ro_mac.img -p 192.168.0.22
tftp -l /dev/mtd6ro -r mtd6ro_reserved2.img -p 192.168.0.22
tftp -l /dev/mtd7ro -r mtd7ro_cfg-param.img -p 192.168.0.22
tftp -l /dev/mtd8ro -r mtd8ro_log.img -p 192.168.0.22
tftp -l /dev/mtd9ro -r mtd9ro_oops.img -p 192.168.0.22
tftp -l /dev/mtd10ro -r mtd10ro_reserved3.img -p 192.168.0.22
tftp -l /dev/mtd11ro -r mtd11ro_web.img -p 192.168.0.22
tftp -l /dev/mtd12ro -r mtd12ro_kernel.img -p 192.168.0.22
tftp -l /dev/mtd13ro -r mtd13ro_rootfs.img -p 192.168.0.22
tftp -l /dev/mtd14ro -r mtd14ro_data.img -p 192.168.0.22
tftp -l /dev/mtd15ro -r mtd15ro_fota.img -p 192.168.0.22
tftp -l /dev/mtd16ro -r mtd16ro_firmware.img -p 192.168.0.22
```

# STEP 3: Booting dari initramfs

- Berbeda dengan STEP 1, kali ini kita perlu ngebreak/meng-interupt `Booting Sequence` dengan cara "tekan sembarang tombol keyboard (USB TTL CP2102)" atau "ctrl+pause break (USB TTL CH340 series)" waktu awal router dihidupkan agar U-Boot tidak meload OS router ini lalu kita bisa menginput command di bawah.
```bash
setenv serverip 192.168.0.22
setenv ipaddr 192.168.0.1
tftpboot 0x81000000 openwrt-23.05.2-ath79-nand-zte_mf286r-initramfs-kernel.bin
bootm 0x81000000
```
- Ketika kita sudah berhasil booting menggunakan `initramfs/ramdisk/mini-os` diperlukan membackup 2 partisi lagi
```bash
cat /dev/mtd4 > mtd4_kernel.bin
cat /dev/mtd9 > mtd9_ubi.bin
```

# STEP 4: Menginstall OS OpenWrt

- Copy firmware OpenWrt yang berada di folder `Orbit Pro2 - ZTE MF286R - Final-Release` -> `openwrt-23.05.2-ath79-nand-zte_mf286r-squashfs-sysupgrade.bin` ke router menggunakan protokol SCP (Secure Copy), taruh di folder `/tmp`
- Install firmware OpenWrt menggunakan command ini:
```bash
sysupgrade -n /tmp/openwrt-23.05.2-ath79-nand-zte_mf286r-squashfs-sysupgrade.bin
```
- Tunggu sampai proses instalasi selesai, jika sudah maka router akan reboot kemudian masuk ke OS OpenWrt
- Ketika sudah masuk OpenWrt, restore settingan firmware `backup-OrbitPro2-2024-02-22.tar.gz` menggunakan menu openwrt (System->Backup/Flash Firmware->Restore->Upload Archive->pilih file tar.gz di atas) tunggu sampai router reboot. Selesai

STEP 5: OPSIONAL
================
- Cek IMEI yang sedang dipakai:
```bash
AT*MRD_IMEI?
```
- Ubah IMEI Menggunakan Command di Bawah Secara Berurutan:
```bash
AT*PROD=2
AT*MRD_IMEI=D
AT*MRD_IMEI=W,0101,01NOV2012,867193047568873
AT*PROD=0
AT*MODEMRESET
AT+ZRESET
```
- Tunggu sampai Router reboot, kemudian gunakan command cek IMEI lagi untuk memastikan apakah sudah benar berubah

### Credit:
- [DBAI](https://dbai.team/discord)
- [Thread - Link Firmware](https://discord.com/channels/1127928183824597032/1210095423201611827)