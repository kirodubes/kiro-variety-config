# Changelog

## 2026.10.05

### What Changed
- Variety now changes the wallpaper on the DankMaterialShell editions (kiro-hyprland-dms, kiro-niri-dms). DMS draws
  its own full-screen wallpaper layer, and `set_wallpaper_kiro` had no DMS case: on Hyprland it fell back to swaybg
  (not installed, then `swaymsg`, which does nothing there), so Variety kept downloading wallpapers that never
  appeared. Found on a bare-metal kiro-hyprland-dms install.

- Variety's wallpaper change no longer times out on non-XFCE desktops that also have XFCE installed. The XFCE part
  of `set_wallpaper_kiro` ran on every desktop whenever `xfconf-query` existed. On the bare-metal test box,
  xfce4-desktop held 412 backdrop keys, so the loop made ~1,200 `xfconf-query` calls and took ~11s. Variety kills
  the script after 10s (`Timeout while running set_wallpaper script, killed`), so the wallpaper never changed.

### Technical Details
- The Wayland section now hands the wallpaper to DMS with `dms ipc call wallpaper set "$WP"` when DMS is running,
  right after the noctalia-shell case.
- DMS is detected with `pgrep -u "$(id -u)" -x dms`, not with `dms ipc`. With no DMS running, `dms ipc` unpacks
  DMS's whole embedded UI, and running that next to a starting `dms run` is what made the DMS bar go missing at
  first login (fixed the same day in kiro-hyprland-dms / kiro-niri-dms).
- The niri section skips swaybg when DMS is running, so kiro-niri-dms doesn't start a hidden swaybg under DMS.
- Tested on the bare-metal box: the script set a Variety image through DMS and started no swaybg.

- The XFCE section now runs only when `XDG_CURRENT_DESKTOP` contains `xfce`, matching Variety's stock script. On the
  test box the run went from 11.2s to 0.09s, and `variety --next` set the wallpaper through DMS without a timeout.

### Files Modified
- `etc/skel/.config/variety/scripts/set_wallpaper_kiro`

## 2026.09.17

### What Changed
Fixed `get_wallpaper_kiro` failing on every Variety startup. The script ended with an
unconditional feh tail (`sed "s/\ /\n/g" ~/.fehbg | grep \'`) that upstream ships
**commented out**; our fork had it live. Two consequences:

- On any session without feh (XFCE, Plasma, GNOME — most Kiro editions) `~/.fehbg`
  does not exist, `sed` exits 1, and because it was the last command it became the
  script's exit status. Variety calls this via `subprocess.check_output()`, so every
  startup raised `CalledProcessError` into `variety.log` and History→Back could not
  restore the pre-Variety wallpaper.
- On a feh-driven tiling WM the `elif` chain matched nothing, fell through to the
  gsettings `else`, printed a GNOME URI, and *then* appended feh's answer — two lines
  where Variety expects a single path.

feh is now a proper `elif` branch in the chain (same shape as the existing swaybg
branch), and the script ends with an explicit `exit 0`.

Found by a `/kiro-syscheck` run against a fresh v26.09.17 install in the VM.

### Technical Details
The branch tests `[ -f "$HOME/.fehbg" ]` and parses the quoted path out of feh's saved
restore command with `tr ' ' '\n' | grep \'` — `tr` rather than the old `sed` because
the intent was always a space-to-newline split, not a regex. Placed immediately before
the `else` fallback so every DE-specific branch still wins; a feh box that also has a
DE session keeps its DE answer. The trailing `exit 0` is the load-bearing part: the
script's exit status is otherwise whatever the last branch's command returned, and
several of them (a missing appletsrc, an absent gsettings key) can legitimately fail
while stdout is still useful. Verified all three paths return one line and exit 0:
feh present, feh absent with no DE, and this dev box (chadwm + feh).

### Files Modified
- etc/skel/.config/variety/scripts/get_wallpaper_kiro
- CHANGELOG.md

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
