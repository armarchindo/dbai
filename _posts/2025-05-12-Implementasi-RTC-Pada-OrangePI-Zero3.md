---
title: Implementasi RTC Pada Device Orange PI Zero3 di OpenWRT
tags: [OpenWRT, Linux, RTC]
style: 
color: danger
description: Implementasi RTC Pada Device Orange PI Zero3 di OpenWRT
---

# Implementasi RTC Pada Device Orange PI Zero3 di OpenWRT

# PERSIAPAN 

1. Pastikan menggunakan firmware yang `I2C` pada `DTB/DTS` Kernel yang ON/Menyala, atau juga bisa mengedit `DTB/DTS` pada Kernel anda (Sebagai Percobaan saya menggunakan firmware dari Mbah Wisnu yang ada di [Discord DBAI](https://discord.com/channels/1127928183824597032/1341370129090609162/1341370129090609162) dikarenakan untuk `I2C` sudah ON)
2. Pasang DBAI Docking Board ke Orange PI Zero3

# STEP 1: Install Package Yang Dibutuhkan

```bash
opkg update
opkg install kmod-i2c-gpio-custom i2c-tools
```

# STEP 2: Mengecek RTC di Terminal OpenWRT

- Pastikan Device sudah terpasang DBAI Docking Board
- Buka `Terminal OpenWRT` di Browser atau bisa dengan `SSH`
- Mengecek bahwa `RTC` Tersambung dengan command dibawah
```bash
i2cdetect 1
```
  Jika Tersambung, maka Output dari terminal tersebut akan seperti ini
```
root@A2OS:~# i2cdetect 1
WARNING! This program can confuse your I2C bus, cause data loss and worse!
I will probe file /dev/i2c-1.
I will probe address range 0x08-0x77.
Continue? [Y/n]
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:                         -- -- -- -- -- -- -- --
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
60: -- -- -- -- -- -- -- -- 68 -- -- -- -- -- -- --
70: -- -- -- -- -- -- -- --
```
- Jika sudah terdeteksi seperti diatas, maka kita cek `DUMP` port `I2C` nya
```bash
i2cdump -y 1 0x68
```
  Apabila Output sudah terdeteksi seperti dibawah, maka `RTC` sudah siap digunakan
```
root@A2OS:~# i2cdump -y 1 0x68
No size specified (using byte-data access)
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f    0123456789abcdef
00: 80 00 00 01 01 01 00 b3 b3 95 36 3e e5 da 8f 5d    ?..???.???6>???]
10: 93 df 85 5a 1d bf 56 f7 a5 9d 16 e7 73 56 17 d5    ???Z??V?????sV??
20: d3 4e d3 57 1b 85 c1 dd f4 d1 b4 ad 9f e7 43 37    ?N?W??????????C7
30: c8 eb e7 8e 3f b7 7e d1 75 f5 f6 dd bd 57 4d 95    ??????~?u????WM?
40: 80 00 00 01 01 01 00 b3 b3 95 36 3e e5 da 8f 5d    ?..???.???6>???]
50: 93 df 85 5a 1d 00 56 f7 a5 9d 16 e7 73 56 17 d5    ???Z?.V?????sV??
60: d3 4e d3 57 1b 85 c1 dd f4 d1 b4 ad 9f e7 43 37    ?N?W??????????C7
70: c8 eb e7 8e 3f b7 7e d1 75 f5 f6 dd bd 57 4d 95    ??????~?u????WM?
80: 80 00 00 01 01 01 00 b3 b3 95 36 3e e5 da 8f 5d    ?..???.???6>???]
90: 93 df 85 5a 1d bf 56 f7 a5 9d 16 e7 73 56 17 d5    ???Z??V?????sV??
a0: d3 4e d3 57 1b 85 c1 dd f4 d1 b4 ad 9f e7 43 37    ?N?W??????????C7
b0: c8 eb e7 8e 3f b7 7e d1 75 f5 f6 dd bd 57 4d 95    ??????~?u????WM?
c0: 80 00 00 01 01 01 00 b3 b3 95 36 3e e5 da 8f 5d    ?..???.???6>???]
d0: 93 df 85 5a 1d 00 56 f7 a5 9d 16 e7 73 56 17 d5    ???Z?.V?????sV??
e0: d3 4e d3 57 1b 85 c1 dd f4 d1 b4 ad 9f e7 43 37    ?N?W??????????C7
f0: c8 eb e7 8e 3f b7 7e d1 75 f5 f6 dd bd 57 4d 95    ??????~?u????WM?
```

# STEP 3: Mengkoneksikan RTC ke System di OpenWRT
- Load Module Kernel `RTC` dengan Menjalankan command dibawah
```bash
insmod rtc-ds1307
```
- Jalankan command dibawah untuk mendeskripsikan alamat `RTC` ke system
```bash
echo ds1307 0x68 > /sys/class/i2c-adapter/i2c-1/new_device
```
- Mengecek RTC Sudah terhubung
```bash
ls /dev | grep rtc
```
  Belum Terhubung :
```
root@A2OS:~# ls /dev | grep rtc
rtc0
```
  Terhubung :
```
root@A2OS:~# ls /dev | grep rtc
rtc0
rtc1
```
  `rtc0` : Bawaan CPU/`RTC` Internal
  `rtc1` : RTC Eksternal

# STEP 4: Menggunakan RTC di OpenWRT

- Mengecek waktu pada RTC Eksternal
```bash
hwclock -r -f /dev/rtc1
```
  Apabila Output seperti dibawah, maka RTC tersebut belum tersinkronkan waktunya
```
root@A2OS:~# hwclock -r -f /dev/rtc1
hwclock: ioctl(RTC_RD_TIME) to /dev/rtc1 to read the time failed: Invalid argument
```
- Mensinkronkan waktu `RTC`
  - Buka LuCI, dan menuju ke `System > System`
  - Klik Tombol `Sync with browser`
  - Jika Waktu sudah sinkron, masuk ke terminal OpenWRT atau `SSH`
  - Ketik `hwclock -w -f /dev/rtc1` di terminal, lalu enter
  - Waktu telah tersinkron
- Jika waktu telah tersinkron, maka coba cek lagi waktu `RTC` nya
```bash
hwclock -r -f /dev/rtc1
```
  Apabila Output seperti dibawah, berarti waktu dibawah sudah tersinkron, dan RTC Siap digunakan
```
root@A2OS:~# hwclock -r -f /dev/rtc1
2025-05-13 00:53:20.367789+07:00
```

# STEP 5: Setting RTC Agar Running setelah booting
- Menambahkan file di `/etc/modules.d` agar module `RTC` otomatis ter-load setelah booting
```bash
echo rtc-ds1307 | tee /etc/modules.d/rtc-ds1307
```
- Edit file `/etc/init.d/sysfixtime` dengan menggunakan tinyfilemanager atau FTP
  Sebelum :
```bash
#!/bin/sh /etc/rc.common
# Copyright (C) 2013-2014 OpenWrt.org

START=00
STOP=90

RTC_DEV=/dev/rtc0
HWCLOCK=/sbin/hwclock

boot() {
	hwclock_load
	local maxtime="$(find_max_time)"
	local curtime="$(date +%s)"
	if [ $curtime -lt $maxtime ]; then
		date -s @$maxtime
		hwclock_save
	fi
}

start() {
	hwclock_load
}

stop() {
	hwclock_save
}

hwclock_load() {
	[ -e "$RTC_DEV" ] && [ -e "$HWCLOCK" ] && $HWCLOCK -s -u -f $RTC_DEV
}

hwclock_save(){
	[ -e "$RTC_DEV" ] && [ -e "$HWCLOCK" ] && $HWCLOCK -w -u -f $RTC_DEV && \
		logger -t sysfixtime "saved '$(date)' to $RTC_DEV"
}

find_max_time() {
	local file newest

	for file in $( find /etc -type f ) ; do
		[ -z "$newest" -o "$newest" -ot "$file" ] && newest=$file
	done
	[ "$newest" ] && date -r "$newest" +%s
}
```
  Sesudah :
```bash
#!/bin/sh /etc/rc.common
# Copyright (C) 2013-2014 OpenWrt.org

START=00
STOP=90

RTC_DEV=/dev/rtc1
HWCLOCK=/usr/sbin/hwclock

boot() {

        insmod rtc-ds1307
        echo ds1307 0x68 > /sys/class/i2c-adapter/i2c-1/new_device

        hwclock_load
        $HWCLOCK -s -f $RTC_DEV
        local maxtime="$(find_max_time)"
        local curtime="$(date +%s)"
        #if [ $curtime -lt $maxtime ]; then
        #       date -s @$maxtime
        #       hwclock_save
        #fi
}

start() {
        hwclock_load
}

stop() {
        hwclock_save
}

hwclock_load() {
        [ -e "$RTC_DEV" ] && [ -e "$HWCLOCK" ] && $HWCLOCK -s -u -f $RTC_DEV
}

hwclock_save(){
        [ -e "$RTC_DEV" ] && [ -e "$HWCLOCK" ] && $HWCLOCK -w -u -f $RTC_DEV && \
                logger -t sysfixtime "saved '$(date)' to $RTC_DEV"
}

find_max_time() {
        local file newest

        for file in $( find /etc -type f ) ; do
                [ -z "$newest" -o "$newest" -ot "$file" ] && newest=$file
        done
        [ "$newest" ] && date -r "$newest" +%s
}
```
- Reboot device untuk mengecek bahwa script tersebut berjalan normal

### Credit:
- [DBAI](https://dbai-team.com/discord)
- [Thread - DBAI Docking Board](https://discord.com/channels/1127928183824597032/1349009833944158259)
- [Firmware Mbah Wisnu](https://discord.com/channels/1127928183824597032/1341370129090609162/1341370129090609162)
