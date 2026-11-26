# Stage 2 Kernel & OPlus Modules Cloud CI Kit
**ColorOS 17 (`Nord6port`) -> POCO F7 (`onyx` / SM8735) on Base HyperOS 4 (`OS4.0.4.0.XOLCNXM`)**

Kit repositori GitHub Actions siap-push ini membangun kernel **`6.6.162-ニシュチャイOxy+`** (GKI `android15-6.6-lts`, KMI Gen 8), **`bootmonitor.ko`** (fix `blkdev_put`), first-stage **`zram.ko`** (`hybridswap_zram`), seluruh modul **OPlus `.ko` + 10 stubs + `onyx_compat_nodes.ko`**, serta menyertakan **`rebuild_vendor_dlkm.sh`** dan **`vb_drop_module.py`** di cloud secara gratis — tanpa membebani laptop Windows.

This ready-to-push GitHub Actions repository kit builds the **`6.6.162-ニシュチャイOxy+`** kernel (GKI `android15-6.6-lts`, KMI Gen 8), fixed **`bootmonitor.ko`**, first-stage **`zram.ko`** (`hybridswap_zram`), all **OPlus `.ko` modules + 10 stubs + `onyx_compat_nodes.ko`**, and bundles **`rebuild_vendor_dlkm.sh`** + **`vb_drop_module.py`** on GitHub's `ubuntu-24.04` runners.

---

## Included Stage 2 Patches (`patches/`)

1. **`patches/0001-game-opt-bump-gts-idx-4-for-hos4-mi-reclaim.patch`** *(Mandatory for Base HOS4)*
   - Moves `#define GTS_IDX` from `3` -> **`4`** in `vendor/oplus/kernel/cpu/game_opt/oem_data/gts_common.h`.
   - Reason: Base HyperOS 4 `mi_reclaim.ko` occupies `task_struct->android_oem_data1[3]` (`+0xdd8`). Slot `[4]` (`+0xde0`) is 100% free across all 934 Base HOS4 `.ko`s and all OPlus `.ko`s.
2. **`patches/0002-onyx-compat-nodes-add-cos17-async-perf-ctrl.patch`** *(Recommended for ColorOS 17)*
   - Adds `/proc/oplus_frame_boost_async/async_perf_ctrl` (`0666`, clean no-op `proc_ioctl` + `proc_compat_ioctl` returning `0` for `0xc030fb01`..`0xc030fb08`, 48-byte `_IOWR`, magic `0xFB`) called by COS 17 `libSchedAssistExtImpl.so`.
   - Adds `/proc/minidump_vcpu_stack` (`0440`, read-only stub returning `"0"`) referenced by COS 17 `init.oplus.debug.rc`.

---

## Cara Pakai (3 Langkah Cepat) / 3-Step Quick Start

### 1. Buat Repo GitHub Baru & Push Folder Ini / Create a GitHub Repo & Push
```powershell
cd D:\cos17-onyx\out\stage2_kernel_ci
git init -b main
git add .
git commit -m "Stage 2 Onyx COS17/HOS4 kernel patches & GitHub Actions CI"
git remote add origin https://github.com/<USERNAME>/onyx-cos17-stage2-ci.git
git push -u origin main
```

### 2. Klik "Run workflow" di GitHub Actions / Click "Run workflow"
1. Buka tab **Actions** pada repositori GitHub Anda.
2. Pilih workflow **`Build Stage 2 Onyx COS17 Kernel & OPlus Modules`**.
3. Klik **Run workflow** -> biarkan semua input default (`onyx-v-oss`, `onyx-tools`, `onyx-port`) -> klik **Run workflow**.
4. Tunggu ~12–15 menit hingga build selesai (termasuk verifikasi DWARF `pahole` otomatis via `onyx/check_layouts.py`).

### 3. Download Artifact `Stage2_Onyx_COS17_Kernel_Modules_Bundle.zip`
Di halaman workflow yang sudah selesai (hijau), unduh **`Stage2_Onyx_COS17_Kernel_Modules_Bundle`** dari bagian **Artifacts**. Isi zip:
- `Image` — Kernel `6.6.162-ニシュチャイOxy+` untuk di-pack ke `boot.img` (Header v4).
- `bootmonitor/bootmonitor.ko` — Pengganti `bootmonitor.ko` bawaan Base HOS4 (fix `blkdev_put` holder leak).
- `vendor_boot/zram.ko` — Modul first-stage `hybridswap_zram` untuk dipasang ke `vendor_boot.img` via `tools/vb_drop_module.py --replace lib/modules/zram.ko vendor_boot/zram.ko`.
- `vendor_dlkm/modules/*.ko` + `vendor_dlkm/oplus_manifest.txt` — Seluruh modul OPlus + 10 stubs + `onyx_compat_nodes.ko`, siap digabung ke `vendor_dlkm.img` Base HOS4 via `tools/rebuild_vendor_dlkm.sh`.
- `tools/rebuild_vendor_dlkm.sh` & `tools/vb_drop_module.py` — Script packaging asli dari `origin/onyx-tools`.
- `BUILD_INFO.txt` — Commit SHA, daftar modul, dan checksum SHA-256.

---

## Credits & Acknowledgments

Proyek porting **ColorOS 17 untuk POCO F7 (`onyx`)** ini dibangun di atas karya luar biasa dari para developer berikut — terima kasih sebesar-besarnya atas kontribusi open-source mereka:

- **Nischay ([@nischay12089](https://github.com/nischay12089))** — Penulis kernel `onyx` `6.6.162`, port modul OPlus SM8735, seluruh native HAL/sensor/FOD/vibrator/flashlight bridges, *SwiftFlash*, dan ROM referensi OxygenOS 16 `onyx`:
  - [Xiaomi_Kernel_OpenSource](https://github.com/nischay12089/Xiaomi_Kernel_OpenSource) (`onyx-v-oss`, `onyx-tools`, dll.)
  - [android_kernel_modules_and_devicetree_oneplus_sm8735](https://github.com/nischay12089/android_kernel_modules_and_devicetree_oneplus_sm8735) (`onyx-port`)
  - [Native-cryptoeng-hal-service](https://github.com/nischay12089/Native-cryptoeng-hal-service)
  - [OxygenOS-ColorOS-Flashlight-Fix](https://github.com/nischay12089/OxygenOS-ColorOS-Flashlight-Fix)
  - [SwiftFlash](https://github.com/nischay12089/SwiftFlash)
  - [fodbridge-source](https://github.com/nischay12089/fodbridge-source)
  - [Oplus-vibrator-feature](https://github.com/nischay12089/Oplus-vibrator-feature)
  - [oplus-sensor-bridge](https://github.com/nischay12089/oplus-sensor-bridge)
- **@UnkNwn019 (`@OPN6U`)** — Developer **ColorOS 17 (`v17.0.1.201`, OnePlus 15 `infiniti` -> OnePlus Nord 6 `SM8735`)** yang menjadi base port Android 17 proyek ini (termasuk **Zuyqa** untuk Modded OPlus Launcher).
- **unmoved21 ([@unmoved21](https://github.com/unmoved21/device_xiaomi_onyx-OFRP))** — Maintainer OrangeFox Recovery (`OFOX-2026-11-11-onyx` / `R12.0_1`) untuk POCO F7 (`onyx`).
