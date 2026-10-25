# HDR in games on Ian's PC without giving up XFCE

Research date: 2026-10-25 (UTC), branch `research/hdr-2026-10-24`. Web research only, nothing was tested on the machine.
Machine: Debian 13 (trixie), kernel 6.12, XFCE on Xorg, RTX 3060 Ti, nvidia-open 615.71.10, Acer ED340CU S3 (3440x1440), Steam + GE-Proton11-11, game: Crimson Desert (DX12 → vkd3d-proton).

Labels used below:
- **[source]**: a page or source file I read this session says so (link given).
- **[my calc]**: my own arithmetic or reading, not from a source.
- **[unconfirmed]**: likely, but no source this session proves it for this exact setup.

---

## 1. Plain-language summary

**XFCE on X11 cannot show HDR, and no setting or layer changes that.** Anything that shows HDR has to run as a Wayland compositor (or gamescope) **that owns the screen directly**. You don't have to give up XFCE, though. You can keep it as your everyday desktop and **log out into a separate "game session"** when you want HDR, then log back into XFCE afterwards. Both kinds of session can be installed side by side.

### Ranked options

**WORKS TODAY (best bets)**

1. **A KDE Plasma (Wayland) session used only for HDR gaming, picked at the LightDM login screen.** XFCE stays the default.
   - Debian trixie's KWin 6.3.6 has everything HDR needs: the HDR toggle, the stable `wp_color_manager_v1` protocol and the `frog` protocol **[source: KWin source]**. On NVIDIA it only enables HDR on driver ≥ 565.57.01, and you have 615 **[source: KWin source]**.
   - Your driver (≥ 595.58.03) already has the Vulkan HDR extensions built in. **You do not need `vk-hdr-layer` or `ENABLE_HDR_WSI=1`** **[source: Arch Wiki]**.
   - **Crimson Desert caveat:** under Wine-Wayland (GE-Proton with `PROTON_ENABLE_WAYLAND=1`), the game sends all-zero HDR metadata. On NVIDIA drivers without the fix (anything before **615.78.08** in the 615 branch, or before **595.104.02** in the 595 branch) that makes KWin disconnect it, and the game crashes **[source: Proton #9672, KWin source, NVIDIA changelogs]**. KDE refuses to relax the check (bug 519000 WONTFIX: "the fix … will be for the Nvidia driver") **[source]**. NVIDIA 615.78.08 (7 Oct 2026) lists a fix for exactly this crash path **[source: Phoronix, linuxcompatible]**. Your 615.71.10 is older than that (its display source is identical to 615.71.09 in the files I diffed), and 615.78.08 is **not yet in NVIDIA's Debian 13 repo** **[source: repo index]**. A user also reports 615.78.08 fixed a 615-branch "washed out HDR" regression **[source, §3.10]**. Workarounds are in §3.7; the simplest is option 2 below.
   - One Oct 17 2026 report: RTX 3090 + Plasma 6.8 + GE-Proton11-9 = Crimson Desert HDR working **[source: KDE bug 527216]**. That user had a newer Plasma than Debian's.

2. **Inside that Plasma session, run the game through nested gamescope (`gamescope --hdr-enabled …`).** **On Ian's exact versions (Plasma 6.3.6 + driver 615.71.10) this is the one to try FIRST.** Wine-Wayland direct comes second, until NVIDIA ≥ 615.78.08 reaches the Debian repo.
   - gamescope does its own colour management and talks to KWin over frog/wp. KWin 6.3.6 offers every feature gamescope checks for **[source: both source trees, §3.16]**. KWin 6.3.6 does **not** have the `windows_bt2100` path (added in 6.8) **[source: KWin git, §3.10]**.
   - **Same resolution, same game:** a ProtonDB report (RTX 5070 Ti, NVIDIA 610.57.04) says Crimson Desert "Crashes when enabling in-game HDR without using gamescope" but works with `gamescope -f -W 3440 -w 3440 -H 1440 -h 1440 --hdr-enabled` **[source: ProtonDB, §3.18]**.
   - In 2024 a gamescope collaborator reported that **nested gamescope under Plasma worked with HDR at 3440x1440@144 over DisplayPort on NVIDIA**. The broken case was standalone gamescope, not nested **[source: NVIDIA forum 295314]**. Several NVIDIA users also report more accurate HDR colours via gamescope than via Wine-Wayland **[source: §3.14]**.
   - Turn **NVIDIA Reflex off** in-game (one report crashed with it on under gamescope) **[source: search snippet, §3.11]**.

**MAYBE (works for some, risky on NVIDIA at 3440x1440)**

3. **gamescope as a standalone game session from a spare TTY or a login-screen entry** (Steam Big Picture inside gamescope, no desktop at all).
   - This works well on AMD. On NVIDIA there is a long-open bug: HDR causes corruption or black screens **above 2560x1440@120**, including a 3440x1440 ultrawide report **[source: gamescope #1593, NVIDIA forum 295314]**.
   - An NVIDIA engineer (24 Aug 2026) said the cause is gamescope's scanout buffers not being physically contiguous. NVIDIA plans a driver-side workaround, but there is no date **[source: gamescope #1593]**. The community kernel patch (open-gpu-kernel-modules PR #1305) is **not merged**. It is **not in 615.71.10 or 615.78.08** either: I diffed the source file **[source: NVIDIA source]**.
   - A separate overlay-corruption bug was reproduced on an **RTX 3060 Ti + 615.71.09** (your card). `GAMESCOPE_COMPOSITE_FORCE=1` was reported to help **[source: gamescope #1964]**.
   - Worth one careful try. Keep your expectations low.

4. **Hyprland (0.55.2 in trixie-backports) as the game session.** Hyprland ≥ 0.55 switches into HDR on its own when a fullscreen game asks for it **[source: Arch Wiki]**. It has frequent HDR regressions, though (direct-scanout oversaturation, black screens, washed-out output), and few reports cover NVIDIA RTX 30 **[source: Hyprland GitHub]**. It also needs more manual setup than Plasma. And since GE-Proton 11-9 only reports HDR when the desktop is *already* HDR, Hyprland's "switch to HDR when a game asks" no longer triggers (§3.24).

**NO (not possible or not worth it on this machine today)**

5. **HDR on X11 / XFCE itself:** no. X.org has no plans for HDR **[source: Arch Wiki → X.org issue 1037]**. NVIDIA's 2017 "DeepColor" X extension idea never went anywhere **[source: Phoronix]**.
6. **gamescope nested inside XFCE:** no real HDR. gamescope automatically uses its SDL backend on X11. That backend hard-codes "HDR not active" and outputs SDR (BT.709 / gamma 2.2) unless you set a debug flag that forces PQ output whether or not the screen is in HDR mode **[source: gamescope source]**. (Your earlier "invisible/minimised" gamescope window is a separate problem. The closest lead found is gamescope #1662: NVIDIA ≥ 555, a black window then a crash when upscaling/explicit sync is used, including on the SDL backend. It doesn't matter for HDR, because the SDL backend can't do HDR anyway. https://github.com/ValveSoftware/gamescope/issues/1662 **[source, not confirmed as your cause]**)
7. **Any Wayland compositor nested in an X11 window** (KWin, sway, Weston, etc.): no. The X11 window it draws into can't carry HDR.
8. **Vulkan HDR layer under X11:** no. The layer only works with a Wayland compositor that speaks frog or xx/wp colour management **[source: VK_hdr_layer README]**. Your driver doesn't need it anyway.
9. **sway / labwc / niri / GNOME from Debian trixie:** no or poor. Trixie's sway 1.10.1 and labwc 0.8.3 predate their HDR releases (sway 1.12, labwc 0.20) **[source: Debian packages, Phoronix]**. niri has no HDR **[source: niri discussion]**. GNOME 48 lacks scRGB/frog, so gamescope HDR looks washed out, and mutter leaves HDR metadata zeroed **[source: Arch Wiki]**.

**LATER / MAYBE: XFCE itself on Wayland**

- Xfce 4.20 can run on labwc (`startxfce4 --wayland`), and labwc ≥ 0.20 has HDR10. Trixie's labwc 0.8.3 doesn't, though, and NVIDIA support is unverified. See §3.12.

**ALWAYS AN OPTION: dual-boot Windows**

10. **Dual-boot Windows.** It has the most mature HDR (the game's own HDR menu, as intended). Downsides: a second OS to maintain, a disk or partition for it, and probably a separate Denuvo "machine" activation **[unconfirmed]**. Covered in §3.6.

> **Expectation:** Acer's specs for this monitor family say ~250 nits peak and an 8-bit panel with no local dimming. HDR will mostly change colour and the tone curve, with no real "pop" (§3.19).

### What I'd do (short version)

1. Plug the monitor in over **DisplayPort** (see §3.8: HDMI 2.0 likely can't carry 10-bit at 100 Hz).
2. `sudo apt install --no-install-recommends plasma-desktop systemsettings kscreen` (plasma-desktop depends on plasma-workspace → kwin-wayland → Xwayland, and gives you a real desktop shell to launch Steam from; `systemsettings` and `kscreen` hold the Display/HDR page and are only *Recommends*, so name them; `kscreen` depends on `libkscreen-bin`, which provides `kscreen-doctor`; `--no-install-recommends` keeps out `sddm` and `kde-config-gtk-style`, the module that rewrites XFCE's GTK theme files). **Don't** install `sddm`: keep LightDM. At the LightDM login screen choose **"Plasma (Wayland)"** only when gaming.
3. In Plasma: System Settings → Display & Monitor → tick **"Enable HDR"**, then set **"Maximum SDR Brightness"** **clearly below** the panel's peak (labels from KScreen v6.3.6 `kcm/ui/OutputPanel.qml` **[source]**). This is **required**: gamescope ≥ 3.16.18 (Debian's 3.16.22 too) and GE-Proton both treat "SDR brightness = peak" as *no HDR* (§3.23). Plasma 6.3 has no HDR calibration app; you can set the panel's real peak with `kscreen-doctor output.DP-1.maxBrightnessOverride.<nits>` (§3.17).
4. Check gamescope: `gamescope --version` should say 3.16.22 (trixie-backports; trixie main has no gamescope). Then Crimson Desert, first try (Proton version unchanged): `DXVK_HDR=1 gamescope -W 3440 -H 1440 -w 3440 -h 1440 -r 144 -f --hdr-enabled --force-grab-cursor -- %command%`, with Reflex off and the in-game HDR peak set to the panel's nits. Full low-Denuvo-activation order in §3.9.
5. Later, once nvidia-open ≥ 615.78.08 is in NVIDIA's Debian 13 repo: try plain `PROTON_ENABLE_WAYLAND=1 %command%` (Wine-Wayland direct, no gamescope). Alternative today: the **595 production branch** (595.104.02+ has the same HDR-metadata fix; 595.112.07 is already in the Debian 13 repo with a `nvidia-driver-pinning-595` package). That's a driver downgrade, with trade-offs (§3.25).

---

## 2. Elapsed-time and method note

- Start: **2026-10-25 03:38:26 UTC** (`date`).
- End: see §5.
- Arch Wiki pages cited were last edited: HDR 2026-09-11, Gamescope 2026-05-28, KDE 2026-10-22 (MediaWiki API) **[source]**.
- Method: web searches and page fetches, plus reading the actual source code of gamescope (master, Oct 2026), KWin (tag v6.3.6 = Debian trixie), the NVIDIA open kernel modules (615.71.10 / 615.78.08) and vkd3d-proton history. Package versions come from packages.debian.org and NVIDIA's Debian 13 repo index.

---

## 3. Details with sources

### 3.1 Angle 1: Any HDR path on X11? **No.**

- Arch Wiki HDR page (raw text fetched): "HDR is supported by Wayland's Color management and HDR protocol… X.org has no plans to support HDR." It links X.org issue 1037 ("Support HDR", opened 2020-06-03, still open). https://wiki.archlinux.org/title/HDR , https://gitlab.freedesktop.org/xorg/xserver/-/issues/1037 **[source]**
- KDE: "HDR support only works in a Wayland session." https://wiki.archlinux.org/title/KDE#HDR **[source]**
- NVIDIA proposed an X "DeepColor Visual" extension in 2017. Nothing shipped. https://www.phoronix.com/news/X11-DeepColor-Visual-RFC **[source]**
- Collabora (Wayland colour-management authors): "even X11 has not gained support for HDR… nothing really materialized." https://www.collabora.com/news-and-blog/blog/2020/11/19/developing-wayland-color-management-and-high-dynamic-range/ **[source]**
- Theoretical corner case: X can lease a monitor to a Vulkan app (`VK_EXT_acquire_xlib_display`, made for VR headsets). Proton/vkd3d don't use it, and gamescope's DRM backend has no lease support (I grepped the source). NVIDIA leasing has reported failures. https://forums.developer.nvidia.com/t/nvidia-proprietary-non-open-modules-completely-unable-to-acquire-a-drm-lease-on-any-display-server-all-known-nvidia-drivers-any-hardware/341244 **[source; conclusion = not a usable path]**

### 3.2 Angle 2: gamescope as a standalone game session alongside XFCE

How it works: from a free TTY (Ctrl+Alt+F2) or a login-screen entry, gamescope runs with **no** `DISPLAY`/`WAYLAND_DISPLAY`. It then auto-selects its **DRM backend**, the only backend that drives the monitor directly with HDR (gamescope `src/main.cpp`, `auto_select_backend()`: Wayland if `WAYLAND_DISPLAY`, SDL if `DISPLAY`, else DRM). https://github.com/ValveSoftware/gamescope **[source]**

- Arch Wiki recipe: `gamescope --hdr-enabled --hdr-itm-enable … -- steam -gamepadui -pipewire-dmabuf`, or `gamescope-session-plus steam` from a TTY. Tip on the same page: "An AMDGPU is recommended for use with gamescope – NVIDIA is known to have critical issues." https://wiki.archlinux.org/title/HDR **[source]**
- `gamescope-session-plus` (ChimeraOS) is **not packaged in Debian**. On non-Arch distros you copy its files by hand. Config goes in `~/.config/environment.d/gamescope-session-plus.conf` (`SCREEN_WIDTH`, `OUTPUT_CONNECTOR`, `ENABLE_GAMESCOPE_HDR`, `GAMESCOPECMD` …). https://github.com/ChimeraOS/gamescope-session **[source]**
- Debian: gamescope is **not in trixie main**, only in **trixie-backports 3.16.22**; forky/sid have 3.16.24. Upstream tagged **3.17.0 on 2026-10-23**. https://packages.debian.org/search?keywords=gamescope , https://github.com/ValveSoftware/gamescope/tags **[source]**
- **NVIDIA + HDR + >2560x1440 bug (key blocker):**
  - gamescope #1593 is **open** (Oct 2024 → Aug 2026). Reports include a **3440x1440 ultrawide over DP** with `--hdr-itm-enable`. Corruption persists until reboot. Workaround reported: remove `--hdr-itm-enable`, and avoid toggling HDR or resolution mid-session. https://github.com/ValveSoftware/gamescope/issues/1593 **[source]**
  - NVIDIA's cubanismo (24 Aug 2026): the hardware can't scan out non-contiguous buffers. gamescope allocates scanout via Vulkan, not GBM, and won't change that. NVIDIA plans a gamescope-specific driver workaround, with no date. **[source, same issue]**
  - Community patch NVIDIA/open-gpu-kernel-modules PR #1305 ("Honor RMDisableNoncontigAlloc in the PMA vidmem path"): **open, unmerged** (opened 18 Aug 2026). https://github.com/NVIDIA/open-gpu-kernel-modules/pull/1305 **[source]**. I compared `src/nvidia/src/kernel/mem_mgr/video_mem.c` in tags 615.71.10 and 615.78.08: the files are identical and **lack the patch**. **[source, my diff]**
  - NVIDIA forum 295314: corruption above 2560x1440@120 with HDR **in gamescope-session**. NVIDIA filed internal bug 5240452 (Apr 2025). Latest post (Mar 29 2026) says it's still broken. In the same thread, **nested gamescope under Plasma 6 worked with HDR at 3440x1440@144 (DP)**. https://forums.developer.nvidia.com/t/display-modes-above-2560x1440p-120hz-with-hdr-enabled-cause-flickering-corruption-within-gamescope-session/295314 **[source]**
- **Overlay corruption** (embedded mode, NVIDIA): gamescope #1964 is open. On **4 Oct 2026, RTX 3060 Ti + 615.71.09 + gamescope 3.16.30**: `GAMESCOPE_COMPOSITE_FORCE=1` stopped the corruption. https://github.com/ValveSoftware/gamescope/issues/1964 **[source]**
- **Exiting:** "Switch to Desktop" in Steam's gamescope mode may hang or do nothing outside SteamOS. People kill gamescope from another TTY or use `gamescopectl shutdown`. https://github.com/ValveSoftware/gamescope/issues/1984 (via search), plus guides found by search **[source, anecdotal]**
- **Steam runs once per user:** quit Steam in XFCE before starting a Steam gamescope session on another TTY **[unconfirmed; general behaviour]**.
- NVIDIAGamescope fork (nightwatchmaker): experimental, 0 stars, no releases. Not recommended. https://github.com/nightwatchmaker/NVIDIAGamescope **[source]**

**Verdict:** "maybe". It's the lightest option (no desktop to install), but your exact combination of NVIDIA + 3440x1440 + HDR + standalone mode is the case with the most open bug reports.

### 3.3 Angle 3: A Wayland compositor started only for gaming

| Compositor | Debian trixie version | HDR? | NVIDIA notes |
|---|---|---|---|
| **KDE Plasma / KWin** | **6.3.6** (trixie), 6.7.4 (forky) | **Yes.** Has the HDR toggle; implements `wp_color_manager_v1` + `frog-color-management-v1` (KWin v6.3.6 source tree) | KWin enables wide colour gamut and HDR on NVIDIA only if driver ≥ **565.57.01** (`drm_output.cpp`) → 615 OK. Requests max bpc 10 by default; env `KWIN_DRM_PREFER_COLOR_DEPTH` overrides. |
| Hyprland | 0.55.2 (trixie-backports) | Yes; ≥0.55 auto-HDR for fullscreen content | Many HDR regressions in 2026 (direct scanout oversaturation fixed by `render:direct_scanout = 0`; black screens; washed-out output). Few RTX 30 reports. |
| sway | **1.10.1** (trixie); 1.12 in forky | Only from **1.12** (25 May 2026, Vulkan renderer) | 1.12 no longer refuses NVIDIA proprietary; no NVIDIA HDR reports found |
| labwc | **0.8.3** (trixie); 0.20.2 in forky | Only from **0.20** (wlroots 0.20, Vulkan renderer) | none found |
| niri | not checked in Debian | **No** HDR (2025 discussion: "probably not any time in the near future") | – |
| GNOME/mutter | 48 (trixie) | HDR toggle yes, but gamescope HDR washed out (no scRGB/frog), and mutter leaves HDR metadata zeroed (fixed later) | Mutter 50 improved NVIDIA |

Sources:
- KWin source: https://invent.kde.org/plasma/kwin (tag v6.3.6: `src/wayland/protocols/color-management-v1.xml` = `wp_color_manager_v1`; `src/backends/drm/drm_output.cpp` lines ~266-277 and ~378-386) **[source]**
- Debian versions: https://packages.debian.org/search?keywords=kwin-wayland , …=sway , …=hyprland , …=labwc **[source]**
- `plasma-workspace` 4:6.3.6-2 ships `/usr/share/wayland-sessions/plasma.desktop` and `startplasma-wayland` and depends on `kwin-wayland` (which depends on `xwayland`). `sddm` is only a *Recommends* of `kde-plasma-desktop`. https://packages.debian.org/trixie/plasma-workspace , https://packages.debian.org/trixie/amd64/plasma-workspace/filelist , https://packages.debian.org/trixie/kde-plasma-desktop **[source]**
- Hyprland: https://github.com/hyprwm/Hyprland/discussions/13413 , https://github.com/hyprwm/Hyprland/discussions/14843 , https://github.com/hyprwm/Hyprland/issues/15413 **[source]**
- sway 1.12: https://www.phoronix.com/news/Sway-1.12-Released ; labwc 0.20: https://phoronix.com/news/labwc-0.20-compositor ; niri: https://github.com/niri-wm/niri/discussions/2685 **[source]**
- GNOME notes: Arch Wiki HDR page (above) **[source]**

**Why Plasma over the others for Ian:** it's the only one in Debian **stable** that has HDR and is maintained for NVIDIA. It is widely used for Linux HDR gaming, and it has a GUI HDR switch and an SDR-brightness slider.

**Plasma 6.3 limits vs newer Plasma (know before you start):**
- KWin's workaround that makes Windows games' own HDR brightness settings behave as on Windows (commit 957031ce, "add workaround + configuration for Windows HDR apps", Aug 2025) first appears in **6.5.90/6.6.0**. Debian's 6.3.6 lacks it, so in-game peak-brightness sliders may look different than on Windows **[source: KWin git; effect on Crimson Desert unconfirmed]**.
- GE-Proton11-10 reads HDR state from KWin's per-monitor HDR state. GE-Proton #838 (Oct 2026): auto-detection treats the screen as SDR when KWin's reference and max luminance are equal. Fixes: lower the SDR brightness below the max, or set `PROTON_ENABLE_HDR=1` manually. https://github.com/GloriousEggroll/proton-ge-custom/issues/838 **[source]**

**Running Plasma next to XFCE (safety):**
- LightDM lists sessions from `/usr/share/wayland-sessions/`. Debian's wiki: LightDM "can still start a Wayland session". Some users report a re-login-after-logout glitch. https://wiki.debian.org/Wayland , https://forums.bunsenlabs.org/viewtopic.php?id=9102 **[source]**
- **Don't be logged into XFCE and Plasma at the same time as the same user.** The systemd user bus and `XDG_RUNTIME_DIR` are shared, which causes black screens or logouts. One 2026 report describes a second Plasma Wayland session freezing NVIDIA. https://answers.launchpad.net/ubuntu/+source/xrdp/+question/704566 , https://discuss.cachyos.org/t/plasma-login-starts-a-second-wayland-session-for-an-already-logged-in-user-kwin-nvidia-drm-atomic-commit-failure-freezes-system/34420 **[source]** → Log out of XFCE first, then log into Plasma.
- Plasma writes its own config to `~/.config/k*`, `plasma*` and `~/.local/share/k*`. XFCE uses `~/.config/xfce4`, so they don't overlap **[unconfirmed in detail; common knowledge]**. The one known shared-file risk is GTK theme sync. Plasma's GTK module rewrites `~/.config/gtk-3.0/settings.ini` and `~/.gtkrc-2.0`, which XFCE uses too. In Debian that's `kde-config-gtk-style`, only a *Recommends*, so it stays out with `--no-install-recommends` (§3.11) **[source]**. Optional extra isolation: use a separate Linux user, e.g. `gamer`, for the Plasma session. That user needs its own Steam login and library path, and Denuvo might see it as a new machine **[unconfirmed]**.

### 3.4 Angle 4: Nested approaches (compositor in an X11 window)

- gamescope source (master): `CSDLConnector::IsHDRActive()` returns `false`, and `GetNativeColorimetry()` returns BT.709 / gamma 2.2 unless `--hdr-debug-force-output` is set (debug only; the X11 window still goes out as SDR). `src/Backends/SDLBackend.cpp`. **[source]**
- gamescope's **Wayland** backend (nested in a Wayland desktop) does support HDR. It requires the host compositor to offer `frog_color_management_factory_v1`, or `wp_color_manager_v1` with parametric/primaries/luminance/scRGB features + PQ + BT.2020 (`WaylandBackend.cpp`, `SupportsColorManagement()`). KWin 6.3.6 offers frog → OK. **[source]**
- Plasma 6.7.0 broke nested gamescope HDR. gamescope #2221 was fixed by PR #2222 (closed 8 Jul 2026); Plasma 6.7.1 also fixed it. Not relevant to Debian's 6.3.6. https://github.com/ValveSoftware/gamescope/issues/2221 , https://discuss.kde.org/t/gamescope-hdr-on-6-7/47895 **[source]**
- X11 itself can't describe HDR pixels, and Xwayland can't forward colour info for X11 apps. https://gitlab.freedesktop.org/pq/color-and-hdr/-/issues/26 (via search summary) **[source]**

**Verdict:** a Wayland compositor nested inside X11 cannot output HDR. Nesting only works in the other direction (gamescope inside a Wayland desktop).

### 3.5 Angle 5: Vulkan HDR layers (VK_hdr_layer / ENABLE_HDR_WSI)

- README: the layer adds `VK_EXT_swapchain_colorspace` + `VK_EXT_hdr_metadata` **only when the compositor supports frog-color-management-v1 or xx-color-management-v4**. "Mesa 25.1+ implements the color management protocol directly, this Vulkan layer is no longer necessary." Don't use it together with gamescope. https://github.com/Zamundaaa/VK_hdr_layer **[source]**
- Arch Wiki: "NVIDIA users with drivers **before** 595.58.03 must install vk-hdr-layer… nvidia-open 595.58.03 and later include these extensions by default." https://wiki.archlinux.org/title/HDR **[source]**
- → On Ian's 615 driver: **`ENABLE_HDR_WSI=1` does nothing useful.** There's no Debian package for the layer either (not checked in depth). Requires: a Wayland compositor (KWin/Hyprland/sway ≥1.12…). Useless on X11.

### 3.6 Angle 6: Dual-boot Windows (brief)

Pros: the game's native HDR path as designed; no Wine-Wayland/KWin/NVIDIA bugs; NVIDIA's Windows HDR stack is mature.
Cons: a second OS (updates, disk space; your Wolf-Box disk layout would need a partition or second drive); rebooting to play; Denuvo probably treats Windows as another "PC" (the limit is 5 different PCs per day) **[unconfirmed]**; it doesn't match your Linux-first home-lab setup.
Denuvo limit source: the Steam store page itself (fetched 25 Oct 2026): "Incorporates 3rd-party DRM: Denuvo Anti-tamper. 5 different PC within a day machine activation limit". https://store.steampowered.com/app/3321460/ **[source]** (also reported by https://insider-gaming.com/crimson-desert-denuvo-drm/)

### 3.7 Angle 7: Crimson Desert specifics

- The game has native HDR on PC: its Steam store categories include **"HDR available"** (Steam appdetails API, fetched 25 Oct 2026; release 19 Mar 2026) https://store.steampowered.com/app/3321460/ **[source]**. It has an in-game calibration (peak brightness, paper white). Patch 1.15.00 fixed HDR UI colours; 1.12.02 fixed a Photo Mode crash after an HDR toggle. https://www.dsogaming.com/patches/crimson-desert-patch-1-12-02-released-detailed/ , https://thegameswiki.com/crimson-desert/wiki/best-settings (search summaries) **[source]**
- **GE-Proton11-9 (14 Oct 2026):** fixes the game getting stuck after Play / New Game (Wine-Wayland and XWayland). Adds **automatic HDR detection for DX11/DXVK and DX12/VKD3D-Proton with Wine-Wayland or Gamescope**; `PROTON_ENABLE_HDR`/`PROTON_USE_HDR` are no longer required; "HDR must still be enabled in the desktop or Gamescope"; `DXVK_NO_HDR=1` disables it. 11-10: KDE HDR detection via KWin per-monitor state. 11-11: no HDR changes. https://github.com/GloriousEggroll/proton-ge-custom/releases **[source]**
- **The crash to know about:** Proton #9672: under Wine's Wayland driver (GE-Proton, proton-cachyos), Crimson Desert calls `vkSetHdrMetadataEXT` with **all zeros**. NVIDIA's WSI passes this to KWin, and KWin kills the client with "min_lum can't be higher or equal to max_lum". Mesa filters such values; NVIDIA (595/610) did not. Valve closed the issue as "not planned" (report it upstream to Wine). Valve's own Proton (XWayland) is unaffected, but it also gets **no HDR** outside gamescope. https://github.com/ValveSoftware/Proton/issues/9672 **[source]**
  - KWin 6.3.6 has the same check (`colormanagement_v1.cpp` line ~400) → **Debian Plasma will hit it too** **[source: KWin source]**.
  - **NVIDIA 615.78.08 (7 Oct 2026):** "a possible crash fix surrounding Vulkan HDR content in conjunction with the Wayland color-management-v1 protocol". linuxcompatible: "unvalidated VkHdrMetadataEXT values could hit the color-management-v1 protocol… NVIDIA now checks those values". https://www.phoronix.com/news/NVIDIA-615.78.08-Linux , https://www.linuxcompatible.org/story/nvidia-6157808-linux-driver-adds-proton-latency-fixes-wayland-hdr-crash-patches/ **[source; that it fixes Crimson Desert specifically is unconfirmed]**
  - 615.78.08 is **not yet** in NVIDIA's Debian 13 CUDA repo (newest there: 615.71.10-1); `download.nvidia.com` has 615.78.08 files (.run). https://developer.download.nvidia.com/compute/cuda/repos/debian13/x86_64/ **[source]**
  - Other workarounds from the issue: a small Vulkan layer "hdrfix" that clamps the metadata (code posted in a comment, no repo link; reported working with Crimson Desert on NVIDIA 610.57.04); a KWin binary patch (risky; must be redone on every update). Or **run the game via nested gamescope inside Plasma**, which does its own colour management. A ProtonDB user on NVIDIA 610.57.04 had exactly "crashes when enabling in-game HDR without using gamescope", fixed by gamescope at 3440x1440 (§3.18) **[source; not proven to be the same root cause]**.
- Reports: CachyOS thread (Apr 2026; RTX 4090, KDE): HDR crash on enable; only gamescope `--hdr-enabled` started with HDR (then froze); another user had success with `PROTON_ENABLE_WAYLAND=1 PROTON_VKD3D_HEAP=1 VKD3D_CONFIG=descriptor_heap PROTON_ENABLE_HDR=1 ENABLE_HDR_WSI=1`. https://discuss.cachyos.org/t/crimson-desert-activating-hdr-leads-to-crash-to-desktop/27513 **[source]**
- **Oct 17 2026:** RTX 3090, Plasma 6.8, GE-Proton11-9: HDR working; with `PROTON_ENABLE_WAYLAND=0` HDR is lost. http://www.mail-archive.com/kde-bugs-dist@kde.org/msg1220237.html **[source]**
- Not HDR-related but relevant to NVIDIA: Proton #9595 has Xid 109 freezes on RTX 30/40 cards incl. 615.71.09. https://github.com/ValveSoftware/Proton/issues/9595 **[source]**

### 3.8 Angle 8: DP vs HDMI at 3440x1440 for HDR

- Acer lists the ED340CU S3 as HDR10, **8-bit colour**, DP 1.4 (180 Hz) and HDMI 2.0 (100 Hz). The search summary says 250 cd/m² (third-party sheet). This conflicts with the ~400 nit in the brief, so check your unit's sticker or OSD. https://www.acer.com/il-en/monitors/gaming/nitro-ed0/pdp/UM.CE0EE.302 (page 503'd on fetch; values from search snippets) **[source, partly]**
- Bandwidth, my estimate (CVT-RB2-like timing; real EDID timings may be tighter):

  | Mode | 8 bpc | 10 bpc |
  |---|---|---|
  | 100 Hz | ~12.8 Gb/s | ~15.9 Gb/s |
  | 144 Hz | ~18.8 Gb/s | ~23.5 Gb/s |
  | 180 Hz | ~23.9 Gb/s | ~29.8 Gb/s |

  HDMI 2.0 carries ~14.4 Gb/s of data; DP 1.4 HBR3 ~25.9 Gb/s. → **HDMI at 100 Hz: 10-bit likely doesn't fit, so 8-bit only. DP: 10-bit fits up to ~144 Hz; 180 Hz HDR would be 8-bit unless the monitor supports DSC** **[my calc]**.
- KWin doesn't require 10 bpc for HDR (it asks the driver for max bpc 10; HDR depends on HDR_OUTPUT_METADATA + PQ in EDID + colourspace). So HDR may still switch on at 8 bpc, with more banding **[source: KWin source; picture-quality impact = my reading]**. NVIDIA HDMI 10 bpc is on by default since 555 (`nvidia-modeset hdmi_deepcolor`). https://nvidia.com/es-es/drivers/details/228304 (via search) **[source]**
- **DP caveat on the 615 driver branch:** NVIDIA bug **6760883** is a "DisplayPort link never re-established after monitor DPMS sleep" regression from 610.57.04 → 615.71.09. NVIDIA staff filed it on 11 Sep 2026. Several reporters tie it to **DP + DSC** at high bandwidth (4K240, 5120x1440); "DP + no DSC … function correctly" for one reporter. A 12 Oct report still has a resume failure on 615.78.08. https://forums.developer.nvidia.com/t/615-release-feedback-discussion/382815 (posts #40, #92, #115, #117, #167) **[source]**. Whether 615.71.10 fixes it: **unconfirmed**. Ian's 3440x1440 monitor probably needs no DSC at ≤144 Hz 10-bit (§3.8, my calc). If the screen stays black after it sleeps: unplug/replug DP or reboot, and consider turning off the "screen off" timer while gaming. HDMI wasn't affected in those reports.
- **Recommendation:** use **DisplayPort**. Try **144 Hz** for HDR gaming (10-bit), and compare 180 Hz. In Plasma: `kscreen-doctor` / Display settings to pick the refresh rate.

### 3.9 Practical steps, risks, switching back, Denuvo

**Denuvo facts:** 5 different PCs per day. Swapping Proton versions (and `WINE_CPU_TOPOLOGY`) can count as a new activation; you're locked out for 24 h. https://www.gamingonlinux.com/2025/05/denuvo-will-lock-you-out-of-games-on-linux-steamos-steam-deck-if-you-keep-changing-proton-versions/ **[source]**. A Crimson Desert Linux guide: "each version change = one activation". https://steamcommunity.com/sharedfiles/filedetails/?id=3688748480 **[source]**. **Whether `PROTON_ENABLE_WAYLAND`, gamescope or switching compositor counts: unconfirmed.** SteamDeckHQ says launch-option changes don't count; Pirat_Nation says they do (§3.11). Assume each attempt *might* cost one activation, and do at most 2–3 attempts per day.

**Low-activation plan (Plasma route):**
1. Day 0 (no game launches): switch to DP. `sudo apt install --no-install-recommends plasma-desktop systemsettings kscreen` (`systemsettings` and `kscreen` are only *Recommends* of plasma-workspace, so they must be named **[source: packages.debian.org]**; other small KDE bits may still be missing, so add them if a settings page is empty **[unconfirmed]**). Keep LightDM. If apt asks for a default display manager, **choose lightdm**.
2. Log out of XFCE → LightDM → session "Plasma (Wayland)" → enable HDR in Display settings, set SDR brightness **below** max (helps GE-Proton's auto-detect, per #838).
3. Test HDR with something that isn't Denuvo-protected first (e.g. an HDR video in mpv `--vo=gpu-next`, or a non-Denuvo HDR game).
4. Crimson Desert, **attempt 1** (same GE-Proton11-11, so no Proton change): `DXVK_HDR=1 gamescope -W 3440 -H 1440 -w 3440 -h 1440 -r 144 -f --hdr-enabled --force-grab-cursor -- %command%`. This runs nested in Plasma, with no `PROTON_ENABLE_WAYLAND` and no `ENABLE_HDR_WSI`. Turn NVIDIA Reflex off in-game, then enable in-game HDR and set peak brightness to the panel's real nits. (Why first: §3.10, §3.14, §3.18.)
5. **Attempt 2**, only if 4 fails (still no Proton change): `PROTON_ENABLE_WAYLAND=1 %command%` (Wine-Wayland direct). If it crashes when you toggle HDR, with "min_lum" in the `PROTON_LOG=1` log, that's the zero-metadata bug. Stop retrying.
6. Later, once nvidia-open ≥ 615.78.08 reaches the Debian 13 repo, retry attempt 2.
7. Keep the **same GE-Proton build (11-11)** throughout. Changing Proton versions is the documented Denuvo trigger.

**Switching back:** log out of Plasma → LightDM → pick "Xfce Session". Nothing else changes. Note: LightDM is meant to **remember the last session you picked** (stored through AccountsService `XSession`, with `~/.dmrc` as a fallback), so after a gaming session the greeter may pre-select Plasma. Check the session menu before logging in. https://bugzilla.redhat.com/show_bug.cgi?id=1010694 , https://code.edge.launchpad.net/bugs/1282139 **[source, bug-tracker level]** **Uninstall:** `sudo apt purge plasma-desktop plasma-workspace kwin-wayland systemsettings kscreen && sudo apt autoremove` (check the list before confirming).

**If enabling HDR gives a black or frozen Plasma screen:** this was reported on Plasma 6 + NVIDIA in 2024 (EndeavourOS thread, driver 555). Recovery: switch to a TTY or log into XFCE, then delete or edit `~/.config/kwinoutputconfig.json`, which stores KWin's per-output HDR setting. In KWin v6.3.6 the key is `"highDynamicRange": true`; set it to `false` (`src/outputconfigurationstore.cpp`). One user said deleting the file worked. https://forum.endeavouros.com/t/hdr-wayland-nvidia-black-screen-on-login/53974 **[source; old driver]**. Your XFCE session is not affected by that file **[my reading]**.

**Risks to the XFCE install:** low with `--no-install-recommends` and LightDM kept. Possible side effects: extra autostart entries or portals (`xdg-desktop-portal-kde`) also starting in XFCE; Qt theme env from Plasma **[unconfirmed]**. Mitigation: a separate `gamer` user.

**gamescope standalone (if you want to try):** use the trixie-backports gamescope, Ctrl+Alt+F3, log in, quit Steam in XFCE first, then e.g. `GAMESCOPE_COMPOSITE_FORCE=1 gamescope -e -W 3440 -H 1440 -r 144 --hdr-enabled -- steam -gamepadui` (`-e` = `--steam`, Steam integration; with no DISPLAY/WAYLAND_DISPLAY set, gamescope picks its DRM backend on its own; `--backend drm` forces it). **Don't** use `--hdr-itm-enabled` (inverse tone-mapping of SDR; the flag several 1593 reports blame). Back: Ctrl+Alt+F7. LightDM's upstream `lightdm.conf` (1.32.0) shows `#minimum-vt=7` as the default, so XFCE usually sits on VT7 https://raw.githubusercontent.com/canonical/lightdm/1.32.0/data/lightdm.conf **[source; check `loginctl` if it isn't there]**. Kill with `gamescopectl shutdown` or `pkill gamescope` from another TTY. A reboot clears any NVIDIA corruption that persists afterwards.

### 3.10 Late findings (added 04:05 UTC) — these sharpen the Plasma advice

- **Debian's Plasma 6.3.6 lacks KWin's Windows-game HDR paths.** `windows_bt2100` image-description support landed in KWin commit 49592c5593 (19 Feb 2026), first tagged **v6.7.90 / 6.8.0**. The Windows-HDR-app brightness workaround (957031ce) first appears in 6.6.0. KWin 6.3.6 does have `create_windows_scrgb`. **[source: KWin git]**
  - Older Wine-Wayland logged "Wayland compositor doesn't expose windows_bt2100 image description (HDR may look broken)!" on such compositors. The same NVIDIA-forum user suggested nested gamescope at 3440x1440 `--hdr-enabled` as the fix, and said Plasma 6.8 fixes it. https://forums.developer.nvidia.com/t/615-release-feedback-discussion/382815 (post #81, 14 Sep 2026) **[source]**
  - Current GE-Proton patch `0070-winewayland-detect-hdr-without-windows-image-descriptions.patch` downgrades that error. When the compositor lacks windows_bt2100, it "passes the original colorspace to Vulkan WSI" (HDR10 through NVIDIA's WSI). https://github.com/GloriousEggroll/proton-ge-custom/tree/master/patches/wine-hotfixes/em-fixups **[source]**. On KWin 6.3.6 + NVIDIA, HDR10 therefore goes through NVIDIA's Vulkan WSI, which is the path where the zero-metadata crash happens before 615.78.08. **[my reading of the sources]**
  - Patch `0071-winewayland-use-kwin-output-hdr-state.patch` reads KWin's per-output HDR flag over `kde_output_device_v2`. It binds legacy per-device globals at v3 ("Older KWin versions advertise each device directly. v3 adds HDR."). KWin v6.3.6 `src/wayland/outputdevice_v2.cpp` advertises those per-device globals at version 11 and sends `high_dynamic_range`, so the detection path **does** fit Debian's KWin **[source: both source files]**. Both 0070 and 0071 are in the **GE-Proton11-10 and 11-11** tags (0071 is not in 11-9) **[source: raw.githubusercontent per tag]**.
- **KDE won't loosen the luminance check:** KDE bug 519000 ("consider clamping invalid mastering luminance") was RESOLVED WONTFIX on 22 Apr 2026. Xaver Hugl: "The fix for that will be for the Nvidia driver to add the checks that are required by the spec." https://bugs.kde.org/show_bug.cgi?id=519000 **[source]** → this matches NVIDIA's 615.78.08 fix.
- **NVIDIA 615.78.08** is on download.nvidia.com (.run files dated 2026-10-06) but not in the Debian 13 CUDA repo yet. One user (8 Oct 2026, NVIDIA 615 feedback thread post #153) says it "fixed a regression which caused HDR colors to be washed out (introduced in original 615 driver)". That regression would affect Ian's 615.71.10. https://download.nvidia.com/XFree86/Linux-x86_64/615.78.08/ **[source]**. Avoid mixing the .run installer with repo packages; wait for the repo **[advice]**.
- **NVIDIA DRM colour pipeline** (610.43+, `nvidia-drm color_pipeline` option; can blank the screen when enabling HDR on some compositors) needs **Linux 6.19+**. Ian's kernel 6.12 uses the older HDR properties, so this issue does not apply. NVIDIA's own 615.78.08 README (Wayland known issues): "On Linux v6.19 and newer kernels, the nvidia-drm module supports the per-plane DRM color pipeline API… Some Wayland compositors may not yet correctly handle color pipelines… blank screen (e.g. when enabling system HDR)… `modprobe nvidia_drm color_pipeline=0`". https://download.nvidia.com/XFree86/Linux-x86_64/615.78.08/README/wayland-issues.html , https://linuxiac.com/nvidia-610-43-linux-driver-adds-vulkan-and-wayland-improvements/ **[source; "doesn't apply on 6.12" is my reading of the 6.19 condition]**
- **Revised Crimson Desert order on Debian Plasma 6.3.6 + 615.71.10:** try **nested gamescope inside Plasma first** (attempt 1 in §3.9). Use Wine-Wayland direct only after NVIDIA ≥ 615.78.08 lands, or on a newer Plasma.

### 3.11 More late findings (≈04:09 UTC)

- **Crimson Desert at exactly 3440x1440 through gamescope:** a ProtonDB report (seen via search snippet only; it was not among the 37 reports later rendered in §3.18) used `gamescope -W 3440 -H 1440 -r 175 -f --hdr-enabled --force-grab-cursor -- %command%`, which worked "with no other launch options". It **crashed after a few minutes with NVIDIA Reflex Low Latency on**; disabling Reflex fixed it. GPU/compositor not visible in the snippet. https://www.protondb.com/app/3321460 **[source, partial]** → in §3.9 attempt 1, **turn Reflex off in-game**. Crimson Desert ProtonDB summary: Gold, 733 reports (API). **[source]**
- **Don't run two gamescopes at once on nvidia-open:** Bazzite #6089 (18 Oct 2026; RTX 3090, 615.78.08, Plasma 6.7.5) reports a GPU crash (Xid 51/154) when two gamescope instances with realtime priority run at once. https://github.com/ublue-os/bazzite/issues/6089 **[source]** → e.g. don't launch a game with gamescope launch options inside a gamescope Steam session.
- **Denuvo and launch options, conflicting claims:** SteamDeckHQ says "Changing launch options or enabling DLSS/raytracing doesn't count towards the limit", only Proton version changes do. Pirat_Nation says launch-option changes count too. https://x.com/SteamDeckHQ/status/2045192293813203341 , https://x.com/Pirat_Nation/status/2045080075247243495 (both via search snippets) **[source, conflicting → unconfirmed]**
- **Debian gamescope backport** ships the gamescope WSI layer (`/usr/share/vulkan/implicit_layer.d/VkLayer_FROG_gamescope_wsi.x86_64.json`, 64-bit only) that `--hdr-enabled` needs ("needs Gamescope WSI layer enabled for support from clients" — gamescope `--help`). Binaries go in `/usr/games`. It depends on libwlroots-0.18. The backport changelog (19 Apr 2026): "Rebuild for trixie-backports. Disable openvr backend… Revert to wlroots 0.18… Drop patches that fix crashes on wlroots 0.19". There are no HDR-specific Debian patches. https://metadata.ftp-master.debian.org/changelogs/contrib/g/gamescope/gamescope_3.16.22+ds-1~bpo13+1_changelog https://packages.debian.org/trixie-backports/amd64/gamescope/filelist **[source]**
- **XFCE theme safety:** Plasma's GTK sync module rewrites `~/.config/gtk-3.0/settings.ini` / `~/.gtkrc-2.0`, which XFCE also uses (Arch Wiki KDE, XFCE forum reports). In Debian that module is `kde-config-gtk-style`, only a *Recommends* of plasma-desktop, so `--no-install-recommends` avoids it. https://forum.xfce.org/viewtopic.php?id=9804 , https://packages.debian.org/trixie/plasma-desktop **[source]**

### 3.12 Keep XFCE itself, but on Wayland? (extra angle, ≈04:11 UTC)

- Xfce 4.20 has **experimental** Wayland support. `startxfce4 --wayland` from a TTY runs xfce4-session under **labwc** (or Wayfire). Panel, Thunar, settings, xfdesktop etc. work; xfwm4 does not (it's X11-only). https://wiki.xfce.org/releng/wayland_roadmap , https://xfce.org/about/tour **[source]**
- Debian's xfce4-session 4.20.2-2 ships only `/usr/share/xsessions/xfce.desktop`, with no Wayland session file, so this would be a TTY start or a hand-written session file. https://packages.debian.org/trixie/amd64/xfce4-session/filelist **[source]**
- HDR then depends on labwc: **trixie's labwc 0.8.3 has no HDR**. HDR10 arrived in **labwc 0.20.0** (25 May 2026, "HDR10 support when running with the Vulkan renderer option"; needs wlroots ≥ 0.20.1). wlroots 0.20 exists only in forky/sid; trixie has 0.18. https://github.com/labwc/labwc/releases , https://packages.debian.org/search?keywords=libwlroots-0.20-dev **[source]**
- Xfce's own compositor **xfwl4** (announced Jan 2026, Rust + Smithay) is "under heavy development"; no HDR info found. Smithay-based niri has no HDR. https://lwn.net/Articles/1056159/ **[source]**
- **Verdict: "later, maybe".** This is the most "XFCE-like" HDR path, but on trixie it means backporting or building wlroots 0.20 + labwc 0.20 yourself. labwc HDR on NVIDIA is not confirmed anywhere I found. Revisit when Debian forky becomes stable, or if a trixie-backport appears.

### 3.13 Not pursued

- **Windows VM with GPU passthrough:** there's only one GPU, so the Linux desktop would be gone while the VM runs anyway. Denuvo/VM behaviour was not found in sources. → no.
- **HDR game streaming (Sunshine) from this PC:** HDR capture needs KMS capture on Wayland, so it doesn't escape the compositor requirement. https://gamingonlinux.com/2024/03/sunshine-game-streaming-v0220-adds-hdr-on-linux-wayland-nvidia-improvements **[source]**

### 3.14 Pattern across NVIDIA reports: gamescope nested gives the more trustworthy picture (≈04:12 UTC)

- Bazzite #3852 (Jan 2026; Plasma 6.5.4, nvidia-open; Wine-Wayland `PROTON_ENABLE_WAYLAND=1 PROTON_ENABLE_HDR=1`): colours were **washed out** once vk_hdr_layer was removed. "Running games under gamescope gave accurate colors". The maintainers blamed the NVIDIA driver. https://github.com/ublue-os/bazzite/issues/3852 **[source]** (This was before NVIDIA's native Vulkan HDR extensions in 595.58.03, so it may be partly outdated.)
- Garuda forum (Oct 2025; RTX 3090, Plasma 6.4.5, 580.82.09): HDR games washed out under KDE. gamescope `--hdr-enabled` worked for DMC5. https://forum.garudalinux.org/t/washed-out-colors-in-kde-hdr-nvidia-multi-monitor/46117 **[source]**
- NVIDIA 615 feedback thread #81 (Sep 2026): a Wine-Wayland game crashed on KWin. The suggested fix was nested gamescope at 3440x1440 `--hdr-enabled` **[source, see §3.10]**.
- 615.71.x "washed out HDR" regression reportedly fixed in 615.78.08 (§3.10).
- → **For Ian today (Plasma 6.3.6 + 615.71.10): nested gamescope inside Plasma is the first thing to try**, and Wine-Wayland direct second. **[my synthesis]**

Suggested Crimson Desert launch line for that first try (it keeps GE-Proton11-11, so the Proton version doesn't change):
`DXVK_HDR=1 gamescope -W 3440 -H 1440 -w 3440 -h 1440 -r 144 -f --hdr-enabled --force-grab-cursor -- %command%`
(`DXVK_HDR=1` is from the Arch Wiki gamescope recipe; GE-Proton11-9+ auto-detects HDR under gamescope anyway, so it's harmless. Don't set `ENABLE_HDR_WSI`. Turn **NVIDIA Reflex off** in-game, per the ProtonDB report. Match `-r` to the refresh rate Plasma runs at.) **[sources: Arch Wiki, GE release notes, ProtonDB snippet; the exact line is untested]**

### 3.15 How to check HDR is really on (commands, ≈04:13 UTC)

- **Debian packages for these tools (trixie):** `kscreen-doctor` → `libkscreen-bin`; `modetest` → `libdrm-tests`; `edid-decode` → `edid-decode` (packages.debian.org contents search) **[source]**.
- **Desktop (Plasma):** `kscreen-doctor -o` lists outputs. `kscreen-doctor output.DP-1.hdr.enable` / `.hdr.disable`, and `kscreen-doctor output.DP-1.sdr-brightness.300` (allowed 50–10000) are the subcommands in libkscreen v6.3.6 `src/doctor/doctor.cpp`. Replace `DP-1` with the name `-o` shows. https://invent.kde.org/plasma/libkscreen/-/blob/v6.3.6/src/doctor/doctor.cpp **[source]**
- **Does the link carry HDR metadata?** `modetest -c | grep -A6 HDR_OUTPUT_METADATA` (package `libdrm-tests`) while HDR is on; all zeros after `02 00` means metadata left empty (Arch Wiki HDR troubleshooting). EDID check: `edid-decode /sys/class/drm/card*-DP-*/edid | grep -A3 'HDR Static Metadata'` (package `edid-decode`). https://wiki.archlinux.org/title/HDR **[source]**
- **Monitor OSD:** most monitors show an "HDR" badge or switch picture mode when they get PQ. **[general; check the Acer manual]**
- **gamescope:** `--hdr-debug-heatmap` overlays a nits heatmap (gamescope `--help`). `--mangoapp` shows HDR status in MangoHud (Arch Wiki gamescope: "Certain MangoHud configurations, such as displaying … HDR status, require the use of mangoapp with gamescope"). Plain MangoHud inside gamescope is not supported. https://github.com/ValveSoftware/gamescope , https://wiki.archlinux.org/title/Gamescope **[source]**
- **Wine-Wayland:** `PROTON_LOG=1` and look for `waylanddrv` colour-management lines / "min_lum" errors (Proton #9672, NVIDIA thread #81). **[source]**
- **In-game:** Crimson Desert's HDR toggle only appears or works when the DXGI output reports HDR (GE-Proton auto-detection, §3.7). If it stays greyed out, the session isn't passing HDR through. **[my reading of GE notes]**

### 3.16 KWin 6.3.6 protocol features vs. what gamescope needs (≈04:14 UTC)

KWin v6.3.6 `src/wayland/colormanagement_v1.cpp` advertises these features: parametric, extended_target_volume, set_mastering_display_primaries, set_primaries, set_luminances, windows_scrgb. Transfer functions: gamma22, sRGB, **ST2084 PQ**, ext_linear. Primaries: sRGB, **BT.2020**, P3 and others. https://invent.kde.org/plasma/kwin/-/blob/v6.3.6/src/wayland/colormanagement_v1.cpp **[source]**
gamescope's `bSupportsGamescopeColorManagement` check (WaylandBackend.cpp) requires parametric + set_primaries + set_mastering_display_primaries + extended_target_volume + set_luminances + windows_scrgb + PQ + sRGB/BT.2020 primaries, **all of which KWin 6.3.6 offers**. gamescope can also fall back to frog. → Nested gamescope HDR should be fully supported, protocol-wise, on Debian's Plasma. **[source: both files; actual picture on NVIDIA untested]**
Ian's EDID (HDR10 PQ + BT.2020, per the brief) matches KWin's HDR capability test: `HDR_OUTPUT_METADATA` property + `edid()->supportsPQ()` + colourspace allowed (NVIDIA ≥ 565.57.01). **[source: drm_output.cpp]**

### 3.17 What 615.71.10 vs 615.78.08 actually change (source diff, ≈04:15 UTC)

- I diffed these NVIDIA open-kernel-module display files across tags **615.71.09 → 615.71.10 → 615.78.08**: `nvidia-drm-crtc.c`, `nvidia-modeset-linux.c`, `nvkms-hdmi.c`, `nvkms-evo.c`, `nvkms-dpy.c`, `nvkms-modeset.c`, `nvdp-connector-event-sink.cpp`. They have **0 changed lines between 615.71.09 and 615.71.10**, and also between 615.71.10 and 615.78.08 for the files compared. (`nvidia-drm-crtc.c` did change, 3470 → 3528 lines, between 615.71.10 and 615.78.08, but not in HDR or scanout code by keyword.) https://github.com/NVIDIA/open-gpu-kernel-modules (tags) **[source, my diff]**
- → 615.71.10 is a small respin. Expect the 615.71.09 issues (washed-out HDR regression, DP wake bug) to still apply to Ian's driver **[my inference]**. 615.78.08's HDR fixes (metadata validation, washed-out regression) must live in NVIDIA's closed user-space Vulkan driver, which fits the "VkHdrMetadataEXT values" wording **[my inference]**.
- **Brightness tuning on Plasma 6.3.6** (Debian lacks the 6.6+ calibration app): `kscreen-doctor output.DP-1.maxBrightnessOverride.<nits>` (and `maxAverageBrightnessOverride`, `minBrightnessOverride`) exist in libkscreen 6.3.6's `doctor.cpp`. Set them to what the panel really reaches (Acer lists 250 cd/m² typical, the brief says ~400) so KWin tone-maps correctly. In Crimson Desert, set peak brightness to the same number **[source: doctor.cpp; the advice is mine]**.

### 3.18 ProtonDB, read directly (rendered with headless Chromium, ≈04:16 UTC; 37 most-recent reports loaded)

https://www.protondb.com/app/3321460 **[source]**. Relevant NVIDIA HDR entries:
- **RTX 5070 Ti, NVIDIA 610.57.04, Manjaro, proton-cachyos-slr (~3 weeks ago):** "Crashes when enabling in-game HDR without using gamescope… I got it to work with gamescope". Launch line: `PROTON_ENABLE_WAYLAND=1 ENABLE_HDR_WSI=1 gamescope -f -W 3440 -w 3440 -H 1440 -h 1440 --hdr-enabled -- %command%`. **Same 3440x1440 resolution as Ian. This matches the pre-615.78.08 zero-metadata crash, and the gamescope-nested workaround.**
- **RTX 4070 Ti, NVIDIA 615.78.08, Arch, GE-Proton11-7 (~2 weeks ago):** `PROTON_ENABLE_WAYLAND=1 PROTON_ENABLE_HDR=1 %command%`, "Runs very well". Random freezes were fixed by `sysctl kernel.split_lock_mitigate=0` (journal showed split-lock warnings for the game). The report doesn't say outright that HDR worked. Desktop not stated.
- **RTX 4070 Ti SUPER, NVIDIA 615.71.09, CachyOS, Proton-CachyOS:** Wine-Wayland + HDR flags; "Instability: Occasionally — Turning on HDR … I am pretty sure you only need gamescope to turn it on."
- Several AMD reports use `PROTON_ENABLE_WAYLAND=1 DXVK_HDR=1` successfully. One notes: "Since Crimson Desert is a DX12 game, for HDR to appear as an option in-game use: DXVK_HDR=1" (pre-GE-11-9 auto-detection).
- NVIDIA driver spread in the loaded reports: 610.57.04 ×7, 610.43.03 ×4, 615.71.09 ×3, 615.78.08 ×1, 595.84 ×1, 590.48.01 ×1.
- The 3440x1440@175 "disable Reflex" report from the search snippet was **not** among the 37 loaded reports. It remains snippet-only.

**Extra tip from this (not HDR):** if Crimson Desert freezes randomly on an Intel CPU, check `journalctl -k | grep -i 'split lock'`. Ian has a Ryzen 7 3700X; split-lock detection is mainly an Intel thing, so it probably doesn't apply **[my note, unconfirmed for Zen 2]**.

### 3.19 Expectation check on the monitor (≈04:17 UTC)

- Acer's own data sheet for the sibling model **ED340CUJ0** (UM.CE0EE.004, same 34" 3440x1440 VA ED0 family; the PDF was fetched and its text extracted) lists **peak brightness 250 cd/m², colour depth 8 bit, HDR10, HDMI 2.0, DP 1.4**. https://ik.imagekit.io/pimberly/65707b38e31f5a94719d7ba0/586f9c7e/680fe786c3421c9df1110a2e/product-pdf **[source; sibling model, not the S3 itself]**. Search snippets for the S3 page also show 250 cd/m² and 8-bit (§3.8).
- Arch Wiki: displays without local dimming "may provide a suboptimal experience" in HDR. https://wiki.archlinux.org/title/HDR **[source]**
- → **Be realistic:** on a ~250–400 nit edge-lit VA panel, HDR10 mostly gives wider colour and a different tone curve, not bright highlights. It's worth getting working, but compare it honestly with SDR. Some people prefer SDR on such panels. **[my assessment]**

### 3.20 Standalone gamescope on NVIDIA: the fix landscape as of 25 Oct 2026 (≈04:19 UTC)

- **gamescope PR #2348** ("Add GBM Scanout Backend for NVIDIA to fix DRM Corruption", opt-in `drm_nvidia_gbm_scanout`) was **closed unmerged on 8 Sep 2026**. Collaborator: "NVIDIA should just fix this bug, nobody should have to use GBM." https://github.com/ValveSoftware/gamescope/pull/2348 **[source]**
- **CachyOS #1082** (13 Oct 2026; RTX 4070 Ti SUPER, **615.78.08**, gamescope 3.16.25): the corruption still reproduces at 5120x1440@144 HDR in the gamescope session when you toggle HDR or change resolution. `NVreg_RegistryDwords=RMDisableNoncontigAlloc=1` alone does nothing without PR #1305. Bazzite 44 (which ships it) is clean. "Same monitor and mode work in KDE and Windows." https://github.com/CachyOS/linux-cachyos/issues/1082 **[source]**
- **CachyOS forum "Gamescope Session NVIDIA"** (Aug–Sep 2026; RTX 4070 Super / 3090): HDR "broken or unreliable", advice to keep it off. VRR works via `ADAPTIVE_SYNC=1`. Steam needed "disable GPU blocklist". https://discuss.cachyos.org/t/gamescope-session-nvidia/34980 **[source]**
- In the NVIDIA 615 feedback thread (through post #223, 24 Oct 2026), nobody mentions the promised gamescope scanout workaround having shipped. Xid 109 / random-black-screen reports continue on 615.78.08 too (posts #170, #180). https://forums.developer.nvidia.com/t/615-release-feedback-discussion/382815 **[source]**
- → **Standalone gamescope + HDR on stock Debian + NVIDIA 615 at 3440x1440 is still a "maybe, probably not".** The only proven fix (PR #1305 + registry key) means patching and building NVIDIA's open kernel module yourself, which is outside what a stock Debian install should take on. **[my assessment]**

### 3.21 Small practical facts checked late (≈04:20 UTC)

- **LightDM + Plasma Wayland on trixie:** Debian bug #1110151 ("Wayland sessions immediately crash on new Trixie install", Plasma and Xfce Wayland via LightDM, Intel T480, July 2025) was closed in Nov 2025 after the reporter "cannot reproduce the error anymore". https://bugs.debian.org/1110151 **[source]**. The session file in plasma-workspace v6.3.6 is named exactly **"Plasma (Wayland)"**. It runs `plasma-dbus-run-session-if-needed startplasma-wayland`, so it only starts its own D-Bus if none is running. https://invent.kde.org/plasma/plasma-workspace/-/blob/v6.3.6/login-sessions/plasmawayland.desktop.cmake **[source]**. If the "Plasma (Wayland)" entry fails from LightDM: log in on a TTY and run `startplasma-wayland` to see the error. **[advice]**
- **No kernel parameter needed for modeset:** NVIDIA 595.45.04+ "enables the modesetting parameter by default for the nvidia-drm kernel driver". Since 570.86.16, fbdev is on by default with modeset (Hyprland wiki). https://www.phoronix.com/news/NVIDIA-595.45.04-Linux-Beta , https://wiki.hypr.land/Nvidia/ **[source]**. Check with `cat /sys/module/nvidia_drm/parameters/modeset` → `Y`. **[advice]**
- **Valve's own Proton** (Experimental/Hotfix) runs through XWayland. It gets HDR only inside gamescope; GE-Proton / proton-cachyos are the Wine-Wayland builds (Proton #9672 correction). **[source]**

### 3.22 Version availability, checked once more (≈04:23 UTC)

- NVIDIA Debian 13 repo: newest `nvidia-open` is **615.71.10-1 (file dated 2026-09-24)**. 615.78.08 is not there yet. https://developer.download.nvidia.com/compute/cuda/repos/debian13/x86_64/ **[source]**
- **No newer Plasma for trixie:** kwin-wayland isn't in trixie-backports. The Debian Qt/KDE team's trixie plan page lists no Plasma backports beyond 6.3.0 (Feb 2025). https://wiki.debian.org/PkgQtKde/TrixieReleasePlans , https://packages.debian.org/search?keywords=kwin-wayland **[source]**. Plasma ≥ 6.8 (with `windows_bt2100`) means waiting for Debian 14 "forky" (6.7.4 there today) or leaving Debian stable. Neither is recommended here.

### 3.23 IMPORTANT for a low-nit monitor: SDR brightness must be below peak (≈04:25 UTC)

- gamescope commit **7d4e835** (Xaver Hugl, 25 Oct 2025; in tags 3.16.18+, **so in Debian's 3.16.22**) changed nested HDR detection. It now uses `bExposeHDRSupport = cv_hdr_enabled && uMaxTargetLuminance > uReferenceLuminance`, and logs `HDR INFO … uMaxLum … uRefLum … bExposeHDRSupport`. https://github.com/ValveSoftware/gamescope/commit/7d4e835 **[source: git show]**
- GE-Proton's Wine-Wayland uses the same "max > reference" idea when KWin's HDR flag isn't available (#838), and gamescope #2037 users fixed washed-out or SDR-only output by setting **SDR/paper-white slightly below the HDR peak** (e.g. 340 SDR / 390 peak). https://github.com/ValveSoftware/gamescope/issues/2037 , https://github.com/GloriousEggroll/proton-ge-custom/issues/838 **[source]**
- KWin v6.3.6 **default** SDR reference luminance = `clamp(EDID max-average brightness or 200, 200, 500)` nits (`src/outputconfigurationstore.cpp` line ~514; struct default 200 in `src/core/output.h`). https://invent.kde.org/plasma/kwin/-/blob/v6.3.6/src/outputconfigurationstore.cpp **[source]**. If the Acer's EDID gives the same max-average and peak value (common on budget HDR10 monitors **[unconfirmed for this unit]**), the default SDR level equals the peak.
- → **On a ~250–400 nit panel, KWin's SDR brightness can easily end up equal to the peak, and then gamescope silently stays SDR.** In Plasma's Display settings (or `kscreen-doctor output.DP-1.sdr-brightness.<nits>`), set SDR brightness clearly **below** the panel's peak (e.g. peak 400 → SDR ~200–300). Check the gamescope terminal/log for `bExposeHDRSupport: true`. **[my synthesis of the sources]**
- **Where KWin's numbers come from:** KWin v6.3.6 takes the output's peak from the EDID's *desired content max luminance* (`drm_output.cpp`: `.maxPeakBrightness = edid->desiredMaxLuminance()`), unless `maxBrightnessOverride` is set (`core/output.cpp`). To read the EDID values: `edid-decode /sys/class/drm/card*-DP-*/edid | grep -iE 'luminance'`. Then set `maxBrightnessOverride` to the real peak and SDR brightness clearly below it. https://invent.kde.org/plasma/kwin/-/blob/v6.3.6/src/core/output.cpp **[source; command is general edid-decode usage]**

### 3.24 Two more small items (≈04:25 UTC)

- **Hyprland + GE-Proton ≥ 11-9: chicken-and-egg.** A Hyprland user on GE-Proton11-11.1 (#838, 22–24 Oct 2026) found that games get HDR only if the **whole desktop** is already in HDR. Before, Hyprland switched to HDR on its own when a game asked for it. GE now only reports HDR when the output is already HDR. Their workaround is a Hyprland Lua rule that sets `cm` to HDR when a given game window opens; it worked for one game and not another. https://github.com/GloriousEggroll/proton-ge-custom/issues/838 **[source]** → one more reason to prefer Plasma (toggle HDR on for the gaming session and leave it on).
- **RenoDX (ReShade HDR mod) for Crimson Desert:** community guides use it to rework the game's HDR, and some Steam users blamed it for broken HDR. It isn't needed for any path above. Leave it out while testing, since it also changes game files **[source: search summaries of a rutab.net guide and Steam discussions; Linux behaviour unconfirmed]**.

### 3.25 A driver option available TODAY from the Debian repo: the 595 production branch (≈04:26 UTC)

- **NVIDIA 595.104.02** (production/"recommended stable" branch, **23 Sep 2026**): "Fixed an issue where unvalidated VkHdrMetadataEXT values were forwarded to the Wayland color-management-v1 protocol, which could cause a crash", plus the >2 GiB page-table fix. These are the same two fixes 615.78.08 later listed. https://www.gamingonlinux.com/2026/09/nvidia-recommended-stable-driver-595-104-02-released-for-linux/ , https://www.linuxcompatible.org/story/nvidia-linux-driver-59510402-fixes-pagetable-and-wayland-hdr-crashes/ **[source]**
- NVIDIA's Debian 13 repo already has **`nvidia-open 595.112.07-1` (dated 2026-10-20)** and the pinning packages **`nvidia-driver-pinning-595`** / `nvidia-driver-pinning-595.112.07`. 595.112.07 is a later 595-branch build (Linux 595.112.07 / Windows 597.30, released 22 Oct 2026). Its data-center notes list only compute/server fixes and don't mention HDR. https://developer.download.nvidia.com/compute/cuda/repos/debian13/x86_64/ , https://docs.nvidia.com/datacenter/tesla/tesla-release-notes-595-112-07/index.html **[source]**. That 595.112.07 carries 595.104.02's HDR fix is **likely but unconfirmed** (a later build in the same branch).
- **Why consider it:** it should give the Wine-Wayland zero-metadata fix today. It also avoids two 615-only regressions reported in §3.8/§3.10 (washed-out HDR, introduced in 615; DP-wake bug 6760883, 610→615). **[inference from sources]**
- **Costs / risks:** it's a driver *downgrade* (615 → 595). You lose 615 features such as VK_NV_low_latency2 r2/r3 (Reflex in Proton). DKMS rebuilds the module. Crimson Desert Xid 109 freezes were reported on 595.84, 610 and 615 alike (Proton #9595), so no branch is clean. Whether a driver change costs a Denuvo activation is **unknown**. NVIDIA's Debian guide: "To confine and pin the system to track a specific branch or driver version, install the appropriate pinning package" (`apt install nvidia-driver-pinning-<branch>`), then `apt -V install nvidia-open`; branch changes go through `apt dist-upgrade --autoremove --purge nvidia-open`. It gives **no downgrade instructions**, and you'd have to remove the existing `nvidia-driver-pinning-615*` package first. https://docs.nvidia.com/datacenter/tesla/driver-installation-guide/debian.html **[source; the downgrade path itself is unconfirmed]**. Have a TTY or rescue plan ready before trying it.
- **Recommendation:** try **nested gamescope first** (no driver change). Switch to 595.112.07 only if you want Wine-Wayland HDR now, or if the 615 washed-out regression shows up. Otherwise wait for 615.78.08+ in the repo.

### 3.26 Plasma 6.3.5 / 6.3.6 colour fixes Debian already has (≈04:28 UTC)

From KDE's official changelogs, these are in Debian's 6.3.6. 6.3.5: "Wayland/colormanagement: use 203cd/m² reference luminance for Windows scRGB"; "Backends/drm: reduce minimum luminance for SDR". 6.3.6: "Wayland/colormanagement: fix sending target luminance levels" (the values gamescope's HDR check reads, §3.23), "… also round max_fall and max_cll", "Backends/drm: never use DEGAMMA_LUT", and **"Outputconfigurationstore: disable adaptive sync by default"**. https://kde.org/announcements/changelogs/plasma/6/6.3.4-6.3.5/ , https://kde.org/announcements/changelogs/plasma/6/6.3.5-6.3.6/ **[source]**
→ Debian's 6.3.6 is the best 6.3 for HDR. **VRR is off by default**; switch Adaptive Sync on in Display settings if you want it (HDR flicker/signal-loss with VRR on NVIDIA under KDE Wayland has been reported at high refresh rates, e.g. https://forums.developer.nvidia.com/t/nvidia-555-58-4k-120hz-causes-flickering-artifacting-on-x-and-wayland-with-arch/298187 (older 555 driver, via search summary); if you see flicker, turn VRR off first) **[source for default; VRR advice mine]**.

---

## 4. Open questions and unconfirmed items

- Whether NVIDIA 595.112.07 (in the Debian repo now) carries 595.104.02's HDR-metadata fix. It's very likely, since it's a later build in the same branch, but its notes don't say.
- No NVIDIA staff reply in the 610.43.02 "invalid HDR metadata" forum thread (2 posts, Jun–Jul 2026). The fix is known only from the 595.104.02 / 615.78.08 changelogs. https://forums.developer.nvidia.com/t/610-43-02-invalid-hdr-metadata-min-lum-max-lum-passed-through-to-wayland-compositor-unfiltered-causing-client-termination-system-wide-hdr-also/374323
- Whether NVIDIA 615.78.08 fully fixes Crimson Desert's zero-metadata crash on KWin 6.3.
- Whether a Wine-Wayland ↔ XWayland or compositor change costs a Denuvo activation.
- Monitor real peak brightness (250 vs ~400 nit) and DSC support.
- Whether nested gamescope in KWin 6.3.6 on NVIDIA 615 still handles 3440x1440 HDR (the 2024 report was on 555).
- Steam single-instance behaviour across TTY sessions (expected, not sourced).
- **No report found anywhere for Crimson Desert HDR on Debian 13 / Plasma 6.3 specifically.** Every working report found was on a newer Plasma (6.7–6.8), Hyprland, or unstated. Ian would be the first data point.
- Whether NVIDIA's Vulkan WSI on KWin 6.3.6 (no `windows_bt2100`) gives correct HDR tone for Wine-Wayland games. Some NVIDIA users reported washed-out Wine-Wayland HDR on newer Plasma (§3.14).
- Whether Crimson Desert's in-game HDR toggle appears under nested gamescope with GE-Proton11-11 without `DXVK_HDR=1` (auto-detection says yes; untested here).

## 5. Times

- Start: 2026-10-25 03:38:26 UTC
- End: (filled at finish)
