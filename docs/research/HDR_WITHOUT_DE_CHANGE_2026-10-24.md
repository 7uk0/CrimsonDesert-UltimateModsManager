# HDR in games on Ian's Linux PC without giving up XFCE — research notes

> DRAFT / partial progress push. Research started 2026-10-25 03:38:26 UTC. Final version follows.

## Findings so far (with sources)

- X11 has no HDR path; X.org has no plan to support HDR (Arch Wiki HDR page, via search snippet; direct fetch blocked by bot protection): https://wiki.archlinux.org/title/HDR
- NVIDIA's 2017 "DeepColor Visual" X extension proposal never went anywhere: https://www.phoronix.com/news/X11-DeepColor-Visual-RFC
- gamescope source (master, Oct 2026): auto-selects the **SDL backend when only DISPLAY is set (i.e. on X11)**; the SDL backend's `IsHDRActive()` hard-returns `false` and outputs BT.709/gamma 2.2 unless the debug flag `--hdr-debug-force-output` is set. => gamescope nested in XFCE cannot output real HDR. Source: https://github.com/ValveSoftware/gamescope (src/main.cpp `auto_select_backend`, src/Backends/SDLBackend.cpp)
- gamescope NVIDIA HDR corruption issue still OPEN, includes 3440x1440 ultrawide reports; NVIDIA engineer (Aug 24 2026) says driver-side gamescope workaround planned: https://github.com/ValveSoftware/gamescope/issues/1593
- GE-Proton11-9 (Oct 14 2026): Crimson Desert Play-hang fix + automatic HDR detection for DXVK/VKD3D-Proton under Wine-Wayland or gamescope; HDR must still be enabled in desktop/gamescope: https://github.com/GloriousEggroll/proton-ge-custom/releases
- CachyOS forum Crimson Desert HDR thread (Apr 2026): https://discuss.cachyos.org/t/crimson-desert-activating-hdr-leads-to-crash-to-desktop/27513

### Interim additions (≈03:48 UTC)
- KWin source: tag v6.3.6 (Debian trixie's kwin-wayland 4:6.3.6-1) already implements stable `wp_color_manager_v1` + `frog-color-management-v1`; it also rejects mastering metadata with `min_lum >= max_lum` (`src/wayland/colormanagement_v1.cpp`), the exact error that kills Crimson Desert under Wine-Wayland on NVIDIA (https://github.com/ValveSoftware/Proton/issues/9672).
- KWin "Windows HDR games" workaround (commit 957031ce, Aug 2025) first in v6.5.90 / 6.6.0 → NOT in Debian trixie's 6.3.6.
- Arch Wiki HDR (raw text fetched): nvidia-open >= 595.58.03 ships VK_EXT_swapchain_colorspace + VK_EXT_hdr_metadata natively → vk-hdr-layer/ENABLE_HDR_WSI not needed; gamescope must NOT be combined with the layer.
- NVIDIA forum 295314: embedded gamescope-session HDR corrupts above 2560x1440@120 on NVIDIA; nested gamescope under Plasma 6 Wayland worked HDR at 3440x1440@144 DP (2024 report). Still unresolved Mar 2026.
- gamescope #1964: embedded gamescope overlay corruption reproduced on RTX 3060 Ti + 615.71.09 + gamescope 3.16.30 (Oct 4 2026); `GAMESCOPE_COMPOSITE_FORCE=1` reported to help.
