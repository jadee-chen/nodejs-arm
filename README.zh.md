<div align="center">

# Node.js for Android (ARM)

**[ 🇺🇸 English ](README.md)** &nbsp;|&nbsp; **[ 🇮🇩 Bahasa Indonesia ](README.id.md)** &nbsp;|&nbsp; **[ 🇨🇳 简体中文 ](README.zh.md)**

<br/>

[![CI](https://img.shields.io/github/actions/workflow/status/Towartz/nodejs-arm/node-android.yml?branch=main&label=CI&logo=github)](https://github.com/Towartz/nodejs-arm/actions)
[![Latest release](https://img.shields.io/github/v/release/Towartz/nodejs-arm?logo=github&label=最新发布)](https://github.com/Towartz/nodejs-arm/releases)
[![NDK](https://img.shields.io/badge/NDK-r29-blue?logo=android)](https://developer.android.com/ndk)
[![API](https://img.shields.io/badge/API-24%2B-brightgreen?logo=android)](https://developer.android.com/guide/topics/manifest/uses-sdk-element)
[![Node](https://img.shields.io/badge/Node-v26.x%20%7C%20v24.x-339933?logo=nodedotjs)](https://nodejs.org)
[![License](https://img.shields.io/github/license/Towartz/nodejs-arm)](LICENSE)

通过 GitHub Actions CI/CD 自动交叉编译 **Node.js** (v26.x, v24.x LTS) 适用于 **Android ARM64** (`arm64-v8a`) 与 **ARM32** (`armeabi-v7a`)。

生成完全静态链接 libc++ 的独立命令行程序或 `.so` 动态共享库，并包含用于编译 C/C++ 原生插件的完整 N-API 头文件。

</div>

---

## 🌟 核心特性

- 📱 **完美适配 Android 15 & 16**：完全符合 **16 KB 页面对齐** 规范要求 (`-Wl,-z,max-page-size=16384`)。
- 📦 **支持两种构建模式**：
  - **Standalone** (默认)：独立 ELF 可执行程序 (`node`)，完全静态链接 libc++。
  - **Shared**：动态共享库 (`libnode.so` 或 `lib<name>.so`)，便于通过 JNI / `System.loadLibrary()` 嵌入 Android 应用中。
- 🏷️ **发布渠道选择 (Release Channel)**：
  - **Stable** (默认)：自动查询并克隆官方最新的正式稳定发布标签 (例如 `v26.7.0` 或 `v24.19.0`)。
  - **Pre-release**：克隆活跃的开发分支 (`v26.x` / `v24.x`)。
- 🔒 **运行时混淆版本标识**：在执行 `node -v` 时，通过内存中 XOR 解码显示自定义版本横幅，二进制 ELF 文件与头文件中零明文字符泄露。
- ⚡ **现代化工具链**：Android NDK **r29** (Clang 21)，Ubuntu 24.04 主机构建，ThinLTO 并行加速，以及持久化 `ccache` 缓存。

---

## 📊 架构支持与构建状态

| ABI | 架构 | 均衡模式 (JIT + -O3) | 速度模式 (ThinLTO) | 体积模式 (V8 精简模式) | 状态 |
|:---|:---|:---:|:---:|:---:|:---:|
| **arm64-v8a** | 64 位 ARM | ✅ 已验证 | ✅ 已验证 | ✅ 已验证 | 主力 / 生产级 |
| **armeabi-v7a** | 32 位 ARM | ⚠️ 实验性 | ⚠️ 实验性 | ⚠️ 实验性 | 社区支持 |

> [!NOTE]
> `armeabi-v7a` 属于尽力支持性质，原因是上游 V8 存在 32 位交叉编译缺陷 ([nodejs/node#58975](https://github.com/nodejs/node/issues/58975))。32 位分支的失败不会影响 64 位 ARM 的发布。

---

## 📦 发布包内容

每个 ABI 对应一个独立的发布压缩包 (`<name>-v<version>-android-<abi>.zip`)：

| 文件 / 目录 | 说明 |
|---|---|
| `node` | 编译生成的 Node.js CLI 二进制文件 (位置无关可执行文件 `PIE`，已去除调试符号，通过 `-rdynamic` 导出所有符号)。 |
| `lib<name>.so` | *(仅在 `shared` 共享模式下)* 编译生成的共享动态库，用于嵌入 Android 应用。 |
| `libc++_shared.so` | NDK 原始自带的 `libc++_shared.so` 副本，供下游编译和链接原生插件时使用。 |
| `include/node/` | 完整的 Node.js、V8 和 libuv C/C++ 头文件 + `config.gypi`，完全兼容 `node-gyp` 和 `node-addon-api`。 |

---

## ⚙️ 构建配置模式

| 配置模式 | 编译参数 | 说明与适用场景 |
|---|---|---|
| **balanced** *(默认)* | JIT + `-O3` + `-g0` | 最佳的通用运行性能，均衡的内存开销，编译速度快。推荐用于 Baileys、服务端与 CLI 工具。 |
| **speed** | ThinLTO (`-flto=thin`) | 并行链接时优化。JS/加密运算速度提升 5–15%，链接器内存占用极低。 |
| **size** | `--v8-lite-mode` | 针对极度受限的嵌入式环境精简 V8 功能。降低内存占用，但执行速度降低 2–3 倍。 |

---

## 🛠️ 工具链详情

| 组件 | 版本 | 说明 |
|---|---|---|
| **Android NDK** | `r29` | Clang 21 工具链，支持现代化 C++20/C++23 标准。 |
| **构建主机** | `ubuntu-24.04` | Ubuntu 24.04 LTS 环境，配备 GCC 13.2+ 主机工具。 |
| **最低目标 API** | `android-24` | 对应 Android 7.0 (Nougat) 及以上。Node.js v24+ 最低要求 API 24。 |

---

## 🧩 为 Android 应用的补丁

Android 并非 Node.js 官方核心团队正式支持的目标平台 ([nodejs/node#58505](https://github.com/nodejs/node/issues/58505))。CI 会自动应用以下补丁：

- **libuv**：修复 Android 堆栈跟踪、POSIX 时间平台头文件以及 `uv.gyp` 配置。
- **V8 / libc++**：添加 `std::atomic_ref` 兼容实现，处理 NDK libc++ 缺失的原子符号。
- **zlib**：使用 `getauxval(AT_HWCAP)` / `AT_HWCAP2` 替代过时的 `android_getCpuFeatures()` 以检测硬件 CRC32/PMULL。
- **Clang Consteval 修复**：将 `regexp-bytecodes-inl.h` 中的访问器函数从 `consteval` 降级为 `constexpr`，规避 Clang 编译器缺陷。
- **Turboshaft & Wrappers**：修复 Turboshaft 中的两阶段名称查找断言以及 `wrappers-inl.h` 中的 `decltype` 处理。
- **OpenSSL ASM**：修正 `openssl_asm.gypi` 中的 Android ARM64/ARM 目标检测，避免误识别为 x86_64 架构。
- **ThinLTO**：修改 `common.gypi` 让 Clang 使用 `-flto=thin`，避免链接阶段发生内存耗尽与交换抖动。

---

## 🚀 如何触发构建

1. 点击本仓库顶部的 **Actions** 标签页。
2. 选择 **Node.js for Android (ARM)** 工作流。
3. 点击 **Run workflow** 并选择所需参数：
   - **`node_version`**：`v26.x` 或 `v24.x`。
   - **`release_channel`**：`stable` (正式发布标签) 或 `prerelease` (开发分支)。
   - **`build_mode`**：`standalone` (CLI 可执行文件) 或 `shared` (`.so` 共享库)。
   - **`lib_name`**：自定义名称前缀 (例如 `node` $\rightarrow$ `libnode.so` / `node-android-*.zip`)。
   - **`profile`**：`balanced`、`speed` (ThinLTO) 或 `size` (V8 Lite)。

任何推送到 `main` 分支的提交也会自动触发构建，并将成果上传至 GitHub Release。

---

## 📄 许可证

GPLv3 附带补充条款。详见 [LICENSE](LICENSE) 文件。

由 **[21_whiten (Towartz)](https://github.com/Towartz)** 维护。
