<div align="center">

# Node.js untuk Android (ARM)

**[ 🇺🇸 English ](README.md)** &nbsp;|&nbsp; **[ 🇮🇩 Bahasa Indonesia ](README.id.md)** &nbsp;|&nbsp; **[ 🇨🇳 简体中文 ](README.zh.md)**

<br/>

[![CI](https://img.shields.io/github/actions/workflow/status/Towartz/nodejs-arm/node-android.yml?branch=main&label=CI&logo=github)](https://github.com/Towartz/nodejs-arm/actions)
[![Latest release](https://img.shields.io/github/v/release/Towartz/nodejs-arm?logo=github&label=rilis%20terbaru)](https://github.com/Towartz/nodejs-arm/releases)
[![NDK](https://img.shields.io/badge/NDK-r29-blue?logo=android)](https://developer.android.com/ndk)
[![API](https://img.shields.io/badge/API-24%2B-brightgreen?logo=android)](https://developer.android.com/guide/topics/manifest/uses-sdk-element)
[![Node](https://img.shields.io/badge/Node-v26.x%20%7C%20v24.x-339933?logo=nodedotjs)](https://nodejs.org)
[![License](https://img.shields.io/github/license/Towartz/nodejs-arm)](LICENSE)

Otomatisasi CI/CD GitHub Actions untuk kompilasi silang (*cross-compile*) **Node.js** (v26.x, v24.x LTS) untuk target **Android ARM64** (`arm64-v8a`) dan **ARM32** (`armeabi-v7a`).

Menghasilkan binary executable mandiri yang ter-link secara statis dengan libc++, atau shared library (`.so`), lengkap dengan header N-API untuk membangun native addon C/C++.

</div>

---

## 🌟 Fitur Utama

- 📱 **Mendukung Android 15 & 16**: Memenuhi ketentuan **16 KB page-size alignment** (`-Wl,-z,max-page-size=16384`).
- 📦 **Dua Mode Build**:
  - **Standalone** (Default): Binary executable ELF mandiri (`node`), ter-link secara statis terhadap `libc++`.
  - **Shared**: Shared library (`libnode.so` atau `lib<name>.so`) untuk disematkan ke dalam aplikasi Android melalui JNI / `System.loadLibrary()`.
- 🏷️ **Pilihan Saluran Rilis (Release Channel)**:
  - **Stable** (Default): Otomatis mendeteksi dan meng-clone tag rilis resmi stabil terbaru (misal `v26.7.0` atau `v24.19.0`).
  - **Pre-release**: Meng-clone branch pengembangan aktif (`v26.x` / `v24.x`).
- 🔒 **Branding Terobfuskasi Runtime**: Menyisipkan teks branding kustom saat `node -v` menggunakan dekripsi XOR di memori tanpa meninggalkan teks plaintext pada binary ELF maupun berkas header.
- ⚡ **Toolchain Mutakhir**: Android NDK **r29** (Clang 21), host build di Ubuntu 24.04, akselerasi paralel ThinLTO, dan cache `ccache` persisten.

---

## 📊 Arsitektur & Status Build

| ABI | Arsitektur | Mode Balanced (JIT + -O3) | Mode Speed (ThinLTO) | Mode Size (V8 Lite) | Status |
|:---|:---|:---:|:---:|:---:|:---:|
| **arm64-v8a** | 64-bit ARM | ✅ Terverifikasi | ✅ Terverifikasi | ✅ Terverifikasi | Utama / Produksi |
| **armeabi-v7a** | 32-bit ARM | ⚠️ Eksperimental | ⚠️ Eksperimental | ⚠️ Eksperimental | Dukungan Komunitas |

> [!NOTE]
> `armeabi-v7a` bersifat *best-effort* karena adanya isu cross-compilation V8 32-bit upstream ([nodejs/node#58975](https://github.com/nodejs/node/issues/58975)). Kegagalan pada 32-bit tidak akan membatalkan rilis 64-bit ARM.

---

## 📦 Isi Paket Rilis

Setiap berkas arsip rilis (`<name>-v<version>-android-<abi>.zip`) berisi paket mandiri lengkap untuk ABI tersebut:

| Berkas / Direktori | Penjelasan |
|---|---|
| `node` | Binary CLI Node.js (Position-Independent Executable `PIE`, stripped, seluruh simbol diekspor via `-rdynamic`). |
| `lib<name>.so` | *(Hanya pada mode `shared`)* Shared library hasil kompilasi untuk integrasi aplikasi Android. |
| `libc++_shared.so` | Salinan asli NDK `libc++_shared.so` untuk kebutuhan build/link native addon di tahap berikutnya. |
| `include/node/` | Header lengkap Node.js, V8, libuv + `config.gypi` yang kompatibel penuh dengan `node-gyp` dan `node-addon-api`. |

---

## ⚙️ Profil Konfigurasi Build

| Profil | Flag Kompiler | Deskripsi & Penggunaan |
|---|---|---|
| **balanced** *(Default)* | JIT + `-O3` + `-g0` | Performa runtime terbaik, seimbang dalam memori, waktu build cepat. Direkomendasikan untuk Baileys, server, dan CLI. |
| **speed** | ThinLTO (`-flto=thin`) | Optimasi waktu-link paralel. Memberikan eksekusi JS/kripto 5–15% lebih cepat dengan konsumsi RAM linker yang hemat. |
| **size** | `--v8-lite-mode` | Mematikan fitur non-esensial V8 untuk perangkat minim RAM. Menghemat RAM dengan trade-off eksekusi 2–3x lebih lambat. |

---

## 🛠️ Rincian Toolchain

| Komponen | Versi | Catatan |
|---|---|---|
| **Android NDK** | `r29` | Toolchain Clang 21 dengan standar modern C++20/C++23. |
| **Host Build** | `ubuntu-24.04` | Ubuntu 24.04 LTS runner dengan GCC 13.2+ untuk perkakas host. |
| **Minimum API** | `android-24` | Android 7.0 (Nougat) ke atas. Node.js v24+ membutuhkan API minimal 24. |

---

## 🧩 Patch yang Diterapkan untuk Android

Android bukan platform yang didukung secara resmi oleh tim inti Node.js ([nodejs/node#58505](https://github.com/nodejs/node/issues/58505)). Patch berikut diterapkan secara otomatis di CI:

- **libuv**: Penanganan stack trace Android, header waktu POSIX, dan konfigurasi `uv.gyp`.
- **V8 / libc++**: Penambahan polyfill `std::atomic_ref` dan penanganan simbol atomic yang belum tersedia di NDK libc++.
- **zlib**: Mengganti `android_getCpuFeatures()` lama dengan `getauxval(AT_HWCAP)` / `AT_HWCAP2` untuk deteksi hardware CRC32/PMULL.
- **Perbaikan Consteval Clang**: Menurunkan fungsi pembaca di `regexp-bytecodes-inl.h` dari `consteval` ke `constexpr` guna mencegah bug immediate-escalation Clang.
- **Turboshaft & Wrappers**: Memperbaiki assertion pencarian nama dua tahap di Turboshaft dan penggunaan `decltype` pada `wrappers-inl.h`.
- **OpenSSL ASM**: Memperbaiki deteksi target ARM64/ARM Android pada `openssl_asm.gypi` agar tidak salah teridentifikasi sebagai target x86_64.
- **ThinLTO**: Memodifikasi `common.gypi` agar Clang menggunakan `-flto=thin`, mencegah pemborosan RAM monolitik saat linking.

---

## 🚀 Cara Menjalankan Build

1. Buka tab **Actions** di repositori ini.
2. Pilih workflow **Node.js for Android (ARM)**.
3. Klik tombol **Run workflow** dan pilih parameter yang diinginkan:
   - **`node_version`**: `v26.x` atau `v24.x`.
   - **`release_channel`**: `stable` (rilis resmi) atau `prerelease` (branch dev).
   - **`build_mode`**: `standalone` (aplikasi CLI) atau `shared` (library `.so`).
   - **`lib_name`**: Nama kustom library (contoh: `node` $\rightarrow$ `libnode.so` / `node-android-*.zip`).
   - **`profile`**: `balanced`, `speed` (ThinLTO), atau `size` (V8 Lite).

Setiap push ke branch `main` juga akan secara otomatis memicu proses build dan mengunggah hasilnya ke GitHub Release.

---

## 📄 Lisensi

GPLv3 dengan ketentuan tambahan. Lihat berkas [LICENSE](LICENSE) untuk rincian lengkap.

Dikelola oleh **[21_whiten (Towartz)](https://github.com/Towartz)**.
