<div align="center">

# Node.js for Android (ARM)

**[ 🇺🇸 English ](README.md)** &nbsp;|&nbsp; **[ 🇮🇩 Bahasa Indonesia ](README.id.md)** &nbsp;|&nbsp; **[ 🇨🇳 简体中文 ](README.zh.md)**

<br/>

[![CI](https://img.shields.io/github/actions/workflow/status/Towartz/nodejs-arm/node-android.yml?branch=main&label=CI&logo=github)](https://github.com/Towartz/nodejs-arm/actions)
[![Latest release](https://img.shields.io/github/v/release/Towartz/nodejs-arm?logo=github&label=latest%20release)](https://github.com/Towartz/nodejs-arm/releases)
[![NDK](https://img.shields.io/badge/NDK-r29-blue?logo=android)](https://developer.android.com/ndk)
[![API](https://img.shields.io/badge/API-24%2B-brightgreen?logo=android)](https://developer.android.com/guide/topics/manifest/uses-sdk-element)
[![Node](https://img.shields.io/badge/Node-v26.x%20%7C%20v24.x-339933?logo=nodedotjs)](https://nodejs.org)
[![License](https://img.shields.io/github/license/Towartz/nodejs-arm)](LICENSE)

Automated GitHub Actions CI/CD cross-compiling **Node.js** (v26.x, v24.x LTS) for **Android ARM64** (`arm64-v8a`) and **ARM32** (`armeabi-v7a`).

Produces standalone statically linked executable binaries or shared `.so` libraries with libc++ and complete N-API header headers for building native C/C++ addons.

</div>

---

## 🌟 Key Features

- 📱 **Android 15 & 16 Ready**: Fully aligned with **16 KB page-size** requirements (`-Wl,-z,max-page-size=16384`).
- 📦 **Dual Build Modes**:
  - **Standalone** (Default): Standalone ELF executable (`node`), statically linked against `libc++`.
  - **Shared**: Shared library (`libnode.so` or `lib<name>.so`) for embedding into Android apps via JNI / `System.loadLibrary()`.
- 🏷️ **Release Channels**:
  - **Stable** (Default): Automatically queries and builds the latest official release tag (e.g., `v26.7.0` or `v24.19.0`).
  - **Pre-release**: Builds the active bleeding-edge development branch (`v26.x` / `v24.x`).
- 🔒 **Runtime Obfuscated Branding**: Embeds custom runtime branding banners on `node -v` using in-memory XOR decoding without leaking plaintext strings in compiled ELF binaries or headers.
- ⚡ **Optimized Toolchain**: Android NDK **r29** (Clang 21), host build on Ubuntu 24.04, ThinLTO parallel acceleration, and persistent `ccache` object caching.

---

## 📊 Architecture & Build Status

| ABI | Architecture | Balanced Mode (JIT + -O3) | Speed Mode (ThinLTO) | Size Mode (V8 Lite) | Status |
|:---|:---|:---:|:---:|:---:|:---:|
| **arm64-v8a** | 64-bit ARM | ✅ Verified | ✅ Verified | ✅ Verified | Primary / Production |
| **armeabi-v7a** | 32-bit ARM | ⚠️ Experimental | ⚠️ Experimental | ⚠️ Experimental | Community Supported |

> [!NOTE]
> `armeabi-v7a` is community-supported and best-effort due to upstream V8 32-bit cross-compilation quirks ([nodejs/node#58975](https://github.com/nodejs/node/issues/58975)). Build failures on 32-bit do not block 64-bit ARM releases.

---

## 📦 Release Package Contents

Each release asset archive (`<name>-v<version>-android-<abi>.zip`) contains a self-contained bundle for that ABI:

| File / Directory | Description |
|---|---|
| `node` | The compiled Node.js CLI binary (Position-Independent Executable `PIE`, stripped, all symbols exported via `-rdynamic`). |
| `lib<name>.so` | *(Only in `shared` build mode)* The compiled shared library for embedding into Android applications. |
| `libc++_shared.so` | Bundled unmodified NDK `libc++_shared.so` for building and linking native C++ addons downstream. |
| `include/node/` | Complete Node.js, V8, and libuv C/C++ headers + `config.gypi` matching official header layouts for `node-gyp` and `node-addon-api`. |

---

## ⚙️ Build Configuration Profiles

| Profile | Compiler Flags | Description & Use Case |
|---|---|---|
| **balanced** *(Default)* | JIT + `-O3` + `-g0` | Best general runtime performance, balanced memory footprint, fast build times. Recommended for Baileys, servers, and CLI tools. |
| **speed** | ThinLTO (`-flto=thin`) | Link-Time Optimization with parallel code generation. Delivers 5–15% faster JS/crypto execution with low linker RAM overhead. |
| **size** | `--v8-lite-mode` | Stripped V8 features for constrained embedded environments. Reduces RAM consumption at the cost of 2–3x slower execution. |

---

## 🛠️ Toolchain Details

| Component | Version | Notes |
|---|---|---|
| **Android NDK** | `r29` | Clang 21 toolchain with modern C++20/C++23 support. |
| **Build Host** | `ubuntu-24.04` | Ubuntu 24.04 LTS runner with GCC 13.2+ for host tools. |
| **Minimum API** | `android-24` | Android 7.0 (Nougat) and higher. Node.js v24+ requires minimum API 24. |

---

## 🧩 Patches Applied for Android

Android is not an officially supported target platform by the Node.js core team ([nodejs/node#58505](https://github.com/nodejs/node/issues/58505)). The following patches are automatically applied during CI:

- **libuv**: Android stack traces, POSIX time platform headers, and `uv.gyp` config fixes.
- **V8 / libc++**: Added polyfills for `std::atomic_ref` and bypassed missing atomic symbols on NDK libc++.
- **zlib**: Replaced legacy `android_getCpuFeatures()` with `getauxval(AT_HWCAP)` / `AT_HWCAP2` for hardware CRC32/PMULL detection.
- **Clang Consteval Fix**: Downgraded accessor functions in `regexp-bytecodes-inl.h` from `consteval` to `constexpr` to avoid Clang immediate-escalation bugs.
- **Turboshaft & Wrappers**: Fixed two-phase name lookup assertions in Turboshaft and `decltype` handling in `wrappers-inl.h`.
- **OpenSSL ASM**: Corrected Android ARM64/ARM target detection in `openssl_asm.gypi` to prevent misidentifying Android as x86_64.
- **ThinLTO**: Patched `common.gypi` to use `-flto=thin` for Clang, avoiding monolithic memory thrashing during linking.

---

## 🚀 How to Trigger a Build

1. Navigate to the **Actions** tab in this repository.
2. Select the **Node.js for Android (ARM)** workflow.
3. Click **Run workflow** and choose your desired parameters:
   - **`node_version`**: `v26.x` or `v24.x`.
   - **`release_channel`**: `stable` (official release tags) or `prerelease` (dev branches).
   - **`build_mode`**: `standalone` (CLI executable) or `shared` (`.so` library).
   - **`lib_name`**: Custom name prefix (e.g. `node` $\rightarrow$ `libnode.so` / `node-android-*.zip`).
   - **`profile`**: `balanced`, `speed` (ThinLTO), or `size` (V8 Lite).

Every push to the `main` branch also triggers an automated build and attaches assets to a GitHub Release.

---

## 📄 License

GPLv3 with additional terms. See [LICENSE](LICENSE) for details.

Maintained by **[21_whiten (Towartz)](https://github.com/Towartz)**.
