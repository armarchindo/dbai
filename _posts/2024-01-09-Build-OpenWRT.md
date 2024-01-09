---
title: Membuat ROOTFS OpenWRT
tags: [Builder, OpenWRT, Linux]
style: 
color: danger
description: Membuat ROOTFS OpenWRT Untuk ARM Device
---

# Build ROOTFS OpenWRT
## 1. Intro
- Sebelumnya kalau abang-abang suka baca tentang openwrt pasti abang pernah mampir ke web radenku.com dan situ disitu ada pembahasan " Cara membuat Rootfs tanpa Compile dari Source Openwrt / Immortall ", diweb radenku.com dikasih tutorial cara build rootfs dari `ImageBuilder Armvirt`, tapi ngga cuma dari Firmware berbasis Armvirt, bisa juga pakai Firmware berbasi Armsr. Dan tutorialnya sama seperti yang ada di randenku.com cuma yang membedakan itu `PROFILE` dari devicenya . Kalau Armvirt kan `PROFILE` devicenya `DEFAULT`, kalau `Armsr` itu `PROFILE` devicenya `GENERIC`. nah buat abang yang bingung bagaimana cara biar tahu di ImageBuildet itu support buat device apa aja bisa ketik di terminal " make info ", nanto hasilnya keliatan device apa aja yang support di ImageBuilder itu. Jadi nanti abang bisa Build Firmware dari ImageBuilder yang tersedia di Openwrt / Immortal dengan cara ini. Sudah pembahasan ImageBuilder.

### 2. Preparing Feed Repositori
- Sekarang aku bahas Repository Feed yang ada di ImageBuilder, kan di dalam Folder atau Dir ImageBuilder ada file dengan nama ` repositories.conf `, silakan abang buka dengan text editor atau apapun itu yang bisa mengedit ataupun membuka filenya. Di dalam file itu kan nanti ada repositori dari Openwrt / Immortal jadi didalam file ini abang nanti bisa menambahkan custom repositori dari milik web radenku.com dan lain-lain. jadi abang tidak perlu menambahkan file .ipk yang abang download dulu terus dimasukin folder/dir package ya. dan 1 lagi di paling bawah dalam file `repositories.conf` ada ` option check_signature ` nah di depan text itu di tambahkan tanda ` # `  dan hasil akan jadi ` # option check_signature `, INGAT INI UNTUK .IPK YANG FEED REPOSITORI YA .

### 3. Menambahkan Custom Package
- Dan selanjutnya kita bahas cara menambahkan custom package yang belum beredar secara luas. Di dalam Folder/Dir ImageBuilder kan ada Folder/Dir dengan nama ` Packages `, abang bisa Copy/Paste custom package yang abang dari Group, Orang, atau darimanapun ke dalam Firmware dengan cara Copy/Paste custom package kedalam Folder Packages. Contoh ya, Mas Hilman kan ada tuh share package IP Info dan App Status, nah jadi tinggal Copy/paste aja di Folder/Dir Package.

### 4. Menambahkan Script First Boot (Waktu pertama kali booting)
- Selanjutnya jika abang ingin menambahkan script firstboot atau script pas awal booting abang bisa buat file dengan nama `99-uci-default.sh` dan di dalam file itu tinggal abang tambahkan script yang abang mau, seperti Script untuk modem rakitan, auto sync Jam, Mengubah Hostname dan alamat IP Luci contohnya. tapi sebelum itu pastikan abang buat Folder di dalam ImageBuilder dengan nama files ,didalam folder files buat lagi folder etc dan di dalam folder etc buat lagi folder dengan nama `uci-defaults` dan Paste kan file `99-uci-default.sh` ke dalam folder uci-defaults. jadi nanti script ini bakal menjalankan perintah dari script yang abang masukan di dalam folder `99-uci-default.sh`.

### 5. Membuat Firmware dari Imagebuilder
- Selanjutkan tahap membuat firmware dari ImageBuilder. dan ini contoh Command yang harus di jalan di terminal saat membuat Rootfs dari Image Builder.
```bash
make image PROFILE="" PACKAGES="" FILES="files"
```
- di Command itu kan ada `PFOFILE=""` ya kan, gimana biar bisa tau Profilenya dari device yang mau di buat, abang baca lagi Pembahasan #1, jika sudah di baca nanti keliatan Profile devicenya dan sudah memilih Profile devicenya jadi tinggal di buat seperti `PROFILE="generic"` ( cara pilih generic, karena kita kan lagi bahas bagaimana cara membuat Rootfs bukan untuk device lain ).

- dan Command `PACKAGES=""` disini abang bisa menambahkan nama package yang abang ingin masukan/include kedalam Rootfs abang nanti, contoh saya ingin menambahkan package Theme Argon dan Argon Config di kedalam Rootfs, jadi tinggal `PACKAGES="luci-theme-argon luci-app-argon-config"` seperti, kalau abang build Rootfs dari ImageBuilder Openwrt mungkin abang perlu menambahkan package Theme Argon dan Argon Config kedalam folder Packages yang ada di ImageBuilder dikarenakan repositori dari Openwrt tidak ada package Theme Argon dan Argon Config, beda dengan Immortal ,direpositori Immortal sudah ada package Theme Argon dan Argon Config jadi tidak perlu menambahkan package Theme Argon dan Argon Config ke folder Packages yang ada di dalam ImageBuilder. dan biasanya di ImageBuilder ini sudah ada Default Package yang sudah tersedia didalam ImageBuilder jadi abang tinggal menambahkan package yang abang ingin ke dalam command tersebut. dan 1 lagi jika abang ingin mengabaikan default package bisa menambahkan ` - ` di depan nama package tersebut, contoh `dnsmasq`, untuk menggunakan Openclash kita memerlukan `dnsmasq-full`, jika kita menambahkan `dnsmasq-full` dan `dnsmasq` secara bersamaan pasti package `dnsmasq-full` akan gagal terinstal dan solusinya ada adalah dengan cara tidak mengisntal `dnsmasq` kedalam Rootfs tersebut, dengan command `PACKAGES="-dnsmasq dnsmasq-full"` dan dengan command `dnsmasq-full` akan terinstall secara normal a.k.a tidak bentrok dengan `dnsmasq`.

- dan selanjutnya Command `FILES="files"` ini untuk memasukan file yang ada di pembahasan **#4** tetap harus ada ini command ini pada saat membuat rootfs. **INGAT YA HARUS ADA**, jadi jika misalkan abang-abang sudah paham dari pembahasan semua itu abang bisa dengan gampang membuat Rootfs ataupun Firmware untuk device abang dengan cara :
```bash
make image PROFILE="nama Profile Device" PACKAGES="defaultpackages packageyangabangmau" FILES="files"
```
- dan tinggal enter dan tunggu hasilnya. jika sudah selesai build Rootfs / Firmwarenya bisa abang cek di folder bin di ImageBuilder.
- Segitu dulu ya bang, Terima kasih. Maaf-maaf kalau ada salah kata dan pengucapan. dan apabila ada salah step mohon dikoreksi. Terima Kasih Semuanya.

### Credit
> - Zomboy (Novan)
> - [DBAI Discord](https://discord.com/channels/1127928183824597032/1189818026011410472)