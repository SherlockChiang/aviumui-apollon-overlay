# AviumUI 16.2.1 apollon source overlay

This directory contains the project-specific source changes used for the
AviumUI 16.2.1 Android 16 build for Xiaomi Mi 10T / Mi 10T Pro / Redmi K30S
Ultra (`apollon`). It is intentionally a small overlay, not a copy of the
entire Android source tree.

## Upstream base

The build uses AviumUI's `avium-16.2` manifest and LineageOS 23.2 device,
kernel, and vendor projects. The exact base revisions used for the validated
2026-08-17 build are recorded below:

| Project | Revision |
| --- | --- |
| AviumUI manifests | `981823afd5a1c3fcf740cd3b4eeeb61331ca8304` |
| `device/xiaomi/apollon` | `4b9526dbd4c8369fb9362708faae2b63ca905f22` |
| `device/xiaomi/sm8250-common` | `da4ba935256abcc07973fb17d6b7729058fc9f9b` |
| `kernel/xiaomi/sm8250` | `71b13e62f057a649b77fe4062feb73ee72ad609c` |
| `vendor/xiaomi/apollon` | `0f1038ffa11d56c84e65477742df35473e191432` |
| `vendor/xiaomi/sm8250-common` | `b4a9f9ab59190887fee4d882ff10a1a3cc7e7712` |
| `vendor/pixel/gms` | `729ad4d0e9a3c0fe2f7e548fce00bda76190b0fd` |
| `vendor/pixel/clocks` | `e3beb954fd54d82798f34002425ff58f9a115714` |
| `vendor/pixel/sounds` | `3fa4964358186344d12259db6809081def36469f` |
| `frameworks/base` | `95a1d4ff3979ee0abc48feca0577817d3024a436` |
| `build/soong` | `2e5b2070e5c672dce237ef0118e1d7bb43f6773d` |
| `build/blueprint` | `c39c8a4c103f1393f015a5befa7726f0c14c9bc2` |

The upstream repositories remain the source of the unmodified code. Apply the
patches in this directory on top of the matching revisions above.

## Project-specific changes

1. `vendor/extra/product.mk` selects the AviumUI GMS product bundle and sets
   the apollon device metadata.
2. `0001-apollon-gms-overlay.patch` loads that overlay early enough for
   AviumUI's `WITH_GMS` product conditional.
3. `0002-gms-default-enabled.patch` makes a GMS build initialize
   `Settings.Secure gms_enabled` to enabled on a clean install, while keeping
   an existing user's explicit choice.
4. `0003-soong-memory-limits.patch` passes `SOONG_GOMEMLIMIT` and `SOONG_GOGC`
   into the primary Soong process. This is a build-host reproducibility fix,
   not a runtime ROM feature.
5. `0004-blueprint-deterministic-env.patch` sorts generated environment
   assignments for deterministic build actions. This is also build-host-only.
6. `local_manifests/apollo.xml` pins the LineageOS device/kernel/vendor
   projects used by this build.
7. `local_manifests/pixel.xml` selects AviumUI's Pixel clocks, sounds, and GMS
   repositories.

The kernel, device common tree, Xiaomi vendor tree, and GMS repository had no
local tracked source edits in the validated build other than the files listed
above.

## GMS binaries

The `vendor/pixel/gms` repository is an AviumUI upstream repository. The build
workspace contains seven locally extracted proprietary APKs that are marked
untracked by Git. They are deliberately **not** included in this overlay:

```text
common/proprietary/product/app/Maps/Maps.apk
common/proprietary/product/app/Photos/Photos.apk
common/proprietary/product/app/PrebuiltGmail/PrebuiltGmail.apk
common/proprietary/product/priv-app/DevicePersonalizationPrebuiltPixel2024-playstore_aiai_20250306.00_RC10/DevicePersonalizationPrebuiltPixel2024-playstore_aiai_20250306.00_RC10.apk
common/proprietary/product/priv-app/PrebuiltBugle/PrebuiltBugle.apk
common/proprietary/product/priv-app/PrebuiltGmsCoreVic/PrebuiltGmsCoreVic.apk
common/proprietary/product/priv-app/Velvet/Velvet.apk
```

Do not commit or mirror these binaries unless you have the necessary
redistribution rights. Reproduce them through the permitted AviumUI extraction
workflow (see `vendor/pixel/gms/README.md` and its `extract-files.sh` script),
or distribute a vanilla build plus a separately licensed GMS installation
method.

## Applying the overlay

From the Android source root:

```bash
git -C device/xiaomi/apollon apply /path/to/0001-apollon-gms-overlay.patch
git -C frameworks/base apply /path/to/0002-gms-default-enabled.patch
git -C build/soong apply /path/to/0003-soong-memory-limits.patch
git -C build/blueprint apply /path/to/0004-blueprint-deterministic-env.patch
mkdir -p vendor/extra
cp /path/to/vendor/extra/product.mk vendor/extra/product.mk
chmod 0644 vendor/extra/product.mk
```

Copy both files under `local_manifests/` into `.repo/local_manifests/` before
syncing the device-specific and Pixel projects. Then use the normal AviumUI
build flow:

```bash
source build/envsetup.sh
lunch lineage_apollon-bp4a-userdebug
m bacon
```

The memory variables are optional and should be sized for the build host, for
example `SOONG_GOMEMLIMIT=16GiB SOONG_GOGC=30`.

## Runtime build artifact

The validated package was:

```text
AviumUI-16.2.1-apollon-20260817-Unofficial-GMS.zip
SHA-256: 2383599d65a5c11c287bd9e4bf013a204470a9aa2a5c750f29febaed2ede9b34
```

This overlay does not grant permission to redistribute AviumUI, LineageOS,
Xiaomi vendor, or Google proprietary components. Preserve their upstream
licenses and attribution when publishing.
