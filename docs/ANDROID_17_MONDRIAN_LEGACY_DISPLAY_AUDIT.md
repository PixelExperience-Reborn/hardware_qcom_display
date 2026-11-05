# Android 17 PE-Reborn — legacy Qualcomm Display ownership (mondrian)

**2026-11-05 | P2 matrix row 150: `hardware/qcom/display`**

## Exact source/consumer comparison

- Archived PE `seventeen` head: `13fd5ca6ac107e8a457074ec68cc4dc7e1a249d7`. Root `Android.mk` selects legacy board directories through `TARGET_BOARD_PLATFORM` or special aliases; available subtrees include `msm8084`, `msm8226`, `msm8909`, `msm8994`, `msm8996` and `msm8998`. There is **no `taro` subtree**.
- Existing `LF52406/mondrian_android_device_xiaomi_sm8450-common` `lineage-24.0` sets `TARGET_BOARD_PLATFORM := taro`. Its `common.mk` requires `android.hardware.graphics.mapper@4.0-impl-qti-display`, `vendor.qti.hardware.display.allocator-service`, `vendor.qti.hardware.display.composer-service` and `hardware/qcom-caf/sm8450/display/config/snapdragon_color_libs_config.xml`.
- The *other*, device-relevant PE archive `hardware/qcom-caf/sm8450/display` (matrix row 142, head `734b422bccc9129f2f93e6364137b5c068d7bdaa`) actually declares these names in `composer/Android.bp` and `gralloc/Android.bp`. Its `Android.bp` exports `display_headers` and uses `qtidisplay_defaults` / `qti_kernel_headers`. That code is **not yet proven Android17/QSSI17-compatible**, and its presence does not establish valid gralloc/SDM ABI or boot.
- The old PE `hardware/qcom/display` includes older gralloc1-era/legacy display structures for earlier SoCs. Renaming an old module or enabling an old header import risks clashing with the required Taro vendor composer/gralloc service, and may invalidate buffer-handle and VINTF compatibility.
- The AOSP17 reference manifest does **not** include a `hardware/qcom/display` checkout to upgrade directly. Preserve the official current Android17 SurfaceFlinger, `frameworks/native` Graphics, `hardware/interfaces`, bufferqueue and stable interfaces. Device-specific QTI composer/allocator/gralloc remain a *separate* independently validated source and vendor-stack requirement.

## Decision and validation gates

**Preserve the old PE repository and its fixes as historical references, without importing it into the staged Android17 build profile.** The old `Android.mk` does not select a Taro implementation.

1. Use only one coherent Taro composer + allocator + mapper + gralloc + display header source set; reconcile all Soong namespaces, linker/vendor variants and firmware/vendor blobs.
2. Compare the Android17 framework graphics contracts and FCM/VINTF declarations to the installed QTI HIDL/AIDL services. Do not assume a service is invalid solely because it is HIDL; verify its precise version and device matrix.
3. Resolve `qti_kernel_headers` and `display_headers` producers and check kernel display UAPI. This is shared with Thermal and IPACM header ownership; do not add placeholder header modules.
4. Evaluate panel refresh rate, color calibration, WQHD/FHD mode selection, ROI, landscape orientation and video/render stability via supported vendor interfaces, not legacy HWC1 enum changes.
5. Build Android17 Soong product modules, run `checkvintf` and SELinux enforcing/neverallow; then boot and test on physical mondrian. Include suspend/resume, SurfaceFlinger and GPU allocation, DRM/protected content and sustained load.

**Status:** source origin and module owners identified, **no full Android17 build or device validation**, Taro display P2 dependency remains OPEN. Do not modify existing MistOS/LineageOS trees in this audit.

**Next row:** `hardware/qcom/gps`.
