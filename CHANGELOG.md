# Changelog

## 2026.06.28

### What Changed
Completeness audit + fixes of the two custom Variety wallpaper helper scripts so
wallpaper get/set works across every desktop/WM Kiro ships (HQ MASTER_TODO §6).

- **`set_wallpaper_kiro`** — fixed a real bug in the SDDM `SIMPLE_WMS` array: the
  `hypr` and `i3` entries were written with no separator (`"…/hypr""…/i3"`), so they
  concatenated into one bogus path and neither matched under SDDM. Now two separate
  elements.
- **`get_wallpaper_kiro`** — broadened the KDE branch from Plasma 5-only
  (`KDE_SESSION_VERSION == "5"`) to Plasma 5 **and** 6 (`-ge 5`); Kiro ships Wayland
  Plasma 6, so "History → Back" previously couldn't read the current wallpaper there.
- **`get_wallpaper_kiro`** — added a Wayland readback branch: when no DE matched but
  `swaybg` is running (Hyprland / Sway / niri / wayfire — the compositors
  `set_wallpaper_kiro` drives via swaybg), parse the current image back from the
  running swaybg's `-i <path>` argument so History→Back works on those sessions too.

### Technical Details
The KDE appletsrc layout (`plasma-org.kde.plasma.desktop-appletsrc`, `Image=` key) is
identical on Plasma 5 and 6, so the same grep serves both — only the version guard
needed widening, with a `${KDE_SESSION_VERSION:-0}` default to avoid an unbound-var
test. The swaybg readback reads `/proc/<pid>/cmdline` (null-separated), splits on NUL,
and pulls the token after `-i`; first PID only, to avoid the transient second swaybg
during a wallpaper change. Both scripts pass `bash -n`.

### Files Modified
- etc/skel/.config/variety/scripts/set_wallpaper_kiro
- etc/skel/.config/variety/scripts/get_wallpaper_kiro

## 2026.05.24

### What Changed
README project-info URL changed from `https://erikdubois.be` to the canonical
project site `https://kiroproject.be`. Part of an EDU-wide README URL sweep.

### Files Modified
- README.md

## 2026.05.18

### What Changed
Created project scaffolding files (CHANGELOG, CLAUDE, TODO, IDEAS).

### Technical Details
Initial session-start stubs per global CLAUDE.md workflow requirements.

### Files Modified
- CHANGELOG.md (created)
- CLAUDE.md (created)
- TODO.md (created)
- IDEAS.md (created)
