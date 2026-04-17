# Building on Ubuntu / Pop!_OS

Tested on Ubuntu 26.04 LTS. May also work on 24.04 (see [notes below](#ubuntu-2404)).

Several dependencies must be built from source as these distros ship older versions.
An automated build script is available at [`scripts/build-deps.sh`](../scripts/build-deps.sh).

## System packages

```bash
sudo add-apt-repository universe

sudo apt update

sudo apt install meson ninja-build bison python-is-python3 wayland-protocols libwayland-dev \
  libcairo2-dev libxkbcommon-dev libinput-dev libxxhash-dev \
  libluajit-5.1-dev libxcb1-dev xwayland libdrm-dev lua-lgi \
  libevdev-dev libmtdev-dev libwacom-dev \
  libpango1.0-dev gir1.2-pango-1.0 librsvg2-dev \
  libseat-dev libudev-dev libgbm-dev libgles-dev libegl-dev \
  libvulkan-dev glslang-tools liblcms2-dev libxcb-dri3-dev \
  libxcb-present-dev libxcb-render-util0-dev libxcb-shm0-dev \
  libxcb-xfixes0-dev libxcb-xinput-dev libxcb-composite0-dev \
  libxcb-ewmh-dev libxcb-icccm4-dev libxcb-res0-dev libxcb-xkb-dev \
  libffi-dev libexpat1-dev libxml2-dev libliftoff-dev cmake g++-14 \
  hwdata
```

## Dependencies built from source

The following must be built from source because Ubuntu 24.04 packages are too old.
Build them **in this order** (each depends on the previous). Clone each repo,
check out the listed tag, and build/install using the project's standard method
(cmake or meson). Run `sudo ldconfig` after each install.

| # | Repository | Tag | Build system | Notes |
|---|-----------|-----|-------------|-------|
| 1 | [hyprutils](https://github.com/hyprwm/hyprutils) | `v0.10.4` | cmake | Needs GCC 14+ (`-DCMAKE_CXX_COMPILER=g++-14`) |
| 2 | [hyprlang](https://github.com/hyprwm/hyprlang) | `v0.6.8` | cmake | Needs GCC 14+ |
| 3 | [hyprcursor](https://github.com/hyprwm/hyprcursor) | `v0.1.13` | cmake | Needs GCC 14+ |
| 4 | [wayland](https://gitlab.freedesktop.org/wayland/wayland) | `1.25.0` | meson | `-Ddocumentation=false -Dtests=false` |
| 5 | [wayland-protocols](https://gitlab.freedesktop.org/wayland/wayland-protocols) | `1.48` | meson | |
| 6 | [libdrm](https://gitlab.freedesktop.org/mesa/drm) | `libdrm-2.4.131` | meson | |
| 7 | [pixman](https://gitlab.freedesktop.org/pixman/pixman) | `pixman-0.46.4` | meson | |
| 8 | [libdisplay-info](https://gitlab.freedesktop.org/emersion/libdisplay-info) | `0.3.0` | meson | |
| 9 | [libxkbcommon](https://github.com/xkbcommon/libxkbcommon) | `xkbcommon-1.8.1` | meson | Only needed on Ubuntu 24.04 (ships 1.6.0); wlroots 0.20 requires >= 1.8.0. `-Denable-docs=false` |
| 10 | [libinput](https://gitlab.freedesktop.org/libinput/libinput) | `1.28.1` | meson | Only needed on Ubuntu 24.04 (ships 1.25.x, missing `libinput_device_get_id_bustype`). `-Ddebug-gui=false -Dtests=false -Ddocumentation=false` |
| 11 | [wlroots](https://gitlab.freedesktop.org/wlroots/wlroots) | `0.20.0` | meson | `-Dexamples=false`; verify drm-backend is YES |

## Building CwC

```bash
cd cwcwm
meson setup build
ninja -C build
sudo ninja -C build install
```

CwC should now be available in your display manager or by running `cwc` from a TTY.

## Ubuntu 24.04

Building on Ubuntu 24.04 may require changing `c_std=gnu23` to `c_std=gnu2x` in
`meson.build` (see commit `d986cf5`). This has only been tested locally once.

Ubuntu 24.04 ships `libxkbcommon 1.6.0` and `libinput 1.25.x`, both older than
required (wlroots 0.20 needs xkbcommon >= 1.8.0; cwc uses
`libinput_device_get_id_bustype` added in libinput 1.26). The build script
detects these and builds newer versions from source into `/usr/local`
(side-by-side with the system packages, so GNOME/GDM are unaffected).
Pop!_OS 24.04 ships newer versions and skips these steps.

## Default config runtime dependencies

The shipped config ([`defconfig/rc.lua`](../defconfig/rc.lua) and
[`defconfig/oneshot.lua`](../defconfig/oneshot.lua)) autostarts several
programs. Install them or edit the config to remove the ones you don't want:

```bash
sudo apt install kitty waybar swaybg swayidle playerctl copyq \
  brightnessctl flameshot xdg-desktop-portal-wlr
```

Or via the build script:

```bash
./scripts/build-deps.sh --install-config-deps
```

The default config also sets `HYPRCURSOR_THEME=Bibata-Modern-Classic`. That
theme is not packaged in Ubuntu — install it manually from
[Bibata_Cursor](https://github.com/ful1e5/Bibata_Cursor) or edit
`oneshot.lua` to remove the `setenv` call.