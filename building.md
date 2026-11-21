# Building Cutefish from Source

How to build the Cutefish desktop from source and install it on an existing
Linux distribution.

This guide is written for people trying Cutefish early and for AI coding
agents doing the build for them. Commands are complete and run in order;
an agent can follow them step by step.

Cutefish is developed and tested on Debian 13 (trixie). Other distributions
work if they ship the library versions listed under
[Requirements](#requirements); the package names below are Debian's, and
[Other distributions](#other-distributions) explains how to find the
equivalents.

## Components

| Repository | Need | What it is |
|---|---|---|
| [cutefish-toolkit](https://github.com/cutefishos/cutefish-toolkit) | Required | UI toolkit every component is built on |
| [cutefish-framework](https://github.com/cutefishos/cutefish-framework) | Required | Clients of system services, file operations, PDF documents |
| [cutefish-core](https://github.com/cutefishos/cutefish-core) | Required | Session, system services, polkit agent, screen recorder |
| [cutefish-compositor](https://github.com/cutefishos/cutefish-compositor) | Required | Wayland compositor, window manager, menu bar, dock, launcher, lock screen |
| [filemanager](https://github.com/cutefishos/filemanager) | Required | File manager; also draws the desktop and its icons |
| [icons](https://github.com/cutefishos/icons) | Required | Default icon theme |
| [wallpapers](https://github.com/cutefishos/wallpapers) | Required | Default wallpaper |
| [settings](https://github.com/cutefishos/settings) | Recommended | System Settings |
| [terminal](https://github.com/cutefishos/terminal) | Recommended | Terminal |
| [xdg-desktop-portal-cutefish](https://github.com/cutefishos/xdg-desktop-portal-cutefish) | Recommended | File chooser, screen sharing and settings for sandboxed and browser apps |
| [cutefish-dm](https://github.com/cutefishos/cutefish-dm) | Optional | Login screen; skip it if you keep SDDM, GDM or LightDM |
| [qt-plugins](https://github.com/cutefishos/qt-plugins) | Optional | Qt 6 apps follow dark mode, system fonts and icons, open the Cutefish file chooser, and scroll like Cutefish apps |
| [text-editor](https://github.com/cutefishos/text-editor) | Optional | Text editor |
| [video-player](https://github.com/cutefishos/video-player) | Optional | Video player |
| [calendar](https://github.com/cutefishos/calendar) | Optional | Calendar |
| [calculator](https://github.com/cutefishos/calculator) | Optional | Calculator |
| [preview](https://github.com/cutefishos/preview) | Optional | Image and PDF viewer |
| [system-monitor](https://github.com/cutefishos/system-monitor) | Optional | System monitor |
| [cutefish-computer-use](https://github.com/cutefishos/cutefish-computer-use) | Optional | MCP server that lets agents such as Claude Code and Codex see and use the desktop, with your permission |

Without the recommended components the desktop runs, but the Settings and
Terminal icons pinned to the dock open nothing.

## Requirements

- A GPU with an EGL and OpenGL ES 2 driver and kernel modesetting. Virtual
  machines need a 3D-capable display device, such as virtio-gpu with
  virgl.
- systemd and systemd-logind.
- Wayland 1.23 or newer, wayland-protocols 1.39 or newer, PipeWire 0.3.65 or
  newer, CMake 3.22 or newer.
- A C++20 compiler (GCC or Clang) and Clang, which builds Skia.
- Network access before building, to download Skia for cutefish-toolkit,
  PDFium for cutefish-framework, and libghostty-vt with the Zig compiler
  that builds it for terminal.
- Optional components: Qt 6 for qt-plugins, libmpv 0.35 or newer for
  video-player, Rust and Cargo for cutefish-computer-use.
- gettext's `msgfmt` for the translations. Without it everything still
  builds, in English only.

Everything installs into `/usr`. Use `-DCMAKE_INSTALL_PREFIX=/usr` as shown;
other prefixes do not work, because the session and the compositor start
and trust their programs by their `/usr/bin` paths.

## 1. Install the build dependencies

On Debian 13, for the required and recommended components:

```shell
sudo apt update
sudo apt install git cmake ninja-build pkg-config clang python3 gettext curl minisign \
    libwayland-dev libwayland-bin wayland-protocols \
    libegl-dev libgles-dev libgbm-dev libdrm-dev \
    libxkbcommon-dev libinput-dev libseat-dev libudev-dev libsystemd-dev libpam0g-dev \
    libfontconfig-dev libfreetype-dev libharfbuzz-dev libicu-dev libexpat1-dev \
    libpng-dev libjpeg-dev zlib1g-dev libwebp-dev libtiff-dev libheif-dev libjxl-dev \
    liblcms2-dev libopenjp2-7-dev libhwy-dev libabsl-dev \
    libpipewire-0.3-dev libspa-0.2-dev \
    libavcodec-dev libavformat-dev libavutil-dev libswscale-dev libva-dev \
    libxcb1-dev libxcb-composite0-dev libxcb-xfixes0-dev libxcb-shape0-dev libxcb-sync-dev \
    libarchive-dev libfuse3-dev libmtp-dev libsmbclient-dev libssh-dev libzip-dev \
    libcrypt-dev libxcursor-dev libpcre2-dev gtk-update-icon-cache
```

Add these for the optional components you want:

| Component | Packages |
|---|---|
| cutefish-dm | none beyond the list above |
| qt-plugins | `qt6-base-dev qt6-base-private-dev qt6-declarative-dev qt6-declarative-private-dev librsvg2-dev` |
| text-editor | none beyond the list above |
| video-player | `libmpv-dev` |
| calendar | `libcurl4-openssl-dev libical-dev` |
| calculator | none beyond the list above |
| preview | none beyond the list above |
| system-monitor | none beyond the list above |
| cutefish-computer-use | `rustc cargo librust-wayland-client-dev librust-wayland-scanner-dev librust-wayland-protocols-dev librust-wayland-backend-dev librust-serde-json-dev librust-png-dev librust-jpeg-encoder-dev librust-base64-dev librust-bitflags-dev librust-libc-dev librust-memmap2-dev` |

To build the tests as well, add `libgtest-dev` and configure with
`-DBUILD_TESTING=ON` instead of `OFF` in step 3.

## 2. Get the sources

Clone the repositories side by side. Leave out the optional ones you do not
want.

```shell
mkdir -p ~/cutefish && cd ~/cutefish
for repo in cutefish-toolkit cutefish-framework cutefish-core cutefish-compositor \
    filemanager icons wallpapers settings terminal xdg-desktop-portal-cutefish \
    cutefish-dm qt-plugins text-editor video-player calendar calculator preview \
    system-monitor cutefish-computer-use; do
    git clone https://github.com/cutefishos/$repo.git
done
```

cutefish-toolkit builds a pinned revision of Skia and cutefish-framework a
pinned revision of PDFium, which have to be fetched first:

```shell
cutefish-toolkit/scripts/fetch-skia.sh
cutefish-framework/scripts/fetch-pdfium.sh
```

## 3. Build and install

terminal links [libghostty-vt](https://github.com/ghostty-org/ghostty)
1.0 statically, found as the pkg-config module `cutefish-ghostty-vt`. Build
it first, with the Zig 0.17 release it needs; both downloads are checked
against their publishers' minisign keys. Skip this if you leave out
terminal.

```shell
mkdir -p ~/cutefish/ghostty-vt && cd ~/cutefish/ghostty-vt
zig=zig-$(uname -m)-linux-0.17.1
curl -fsSLO https://ziglang.org/download/0.17.1/$zig.tar.xz
curl -fsSLO https://ziglang.org/download/0.17.1/$zig.tar.xz.minisig
minisign -Vm $zig.tar.xz -P RWSGOq2NVecA2UPNdBUZykf1CCb147pkmdtYxgb3Ti+JO/wCYvhbAb/U
tar -xJf $zig.tar.xz
base=https://release.files.ghostty.org/libghostty-vt/1.0.1
for f in libghostty-vt-1.0.1.tar.gz libghostty-vt-1.0.1-deps.tar.gz; do
    curl -fsSLO $base/$f
    curl -fsSLO $base/$f.minisig
    minisign -Vm $f -P RWQlAjJC23149WL2sEpT/l0QKy7hMIFhYdQOFy0Z7z7PbneUgvlsnYcV
    tar -xzf $f
done
cd libghostty-vt-1.0.1
DESTDIR=~/cutefish/ghostty-vt/stage ../$zig/zig build --prefix /usr \
    --prefix-lib-dir lib/cutefish-ghostty-vt --prefix-include-dir include/cutefish-ghostty-vt \
    --system ../libghostty-vt-1.0.1-deps -Doptimize=ReleaseFast
```

Install only the static library, the headers and a pkg-config file of
Cutefish's own, so nothing of it is needed at run time and a distribution's
libghostty-vt can sit beside it:

```shell
cd ~/cutefish/ghostty-vt/stage/usr
sudo install -Dm644 -t /usr/lib/cutefish-ghostty-vt lib/cutefish-ghostty-vt/libghostty-vt.a
sudo cp -r include/cutefish-ghostty-vt /usr/include/
sudo install -d /usr/share/pkgconfig
sudo tee /usr/share/pkgconfig/cutefish-ghostty-vt.pc > /dev/null << 'EOF'
libdir=/usr/lib/cutefish-ghostty-vt
includedir=/usr/include/cutefish-ghostty-vt

Name: cutefish-ghostty-vt
Description: libghostty-vt, static, for building Cutefish
Version: 1.0.1
Cflags: -I${includedir} -DGHOSTTY_STATIC
Libs: -L${libdir} -lghostty-vt -lm
EOF
pkg-config --modversion cutefish-ghostty-vt
```

The last command prints `1.0.1`.

Then build the projects in this order. Each project is installed before the next is
configured, because later projects find earlier ones through their
installed CMake packages.

```shell
cd ~/cutefish
for repo in cutefish-toolkit cutefish-framework qt-plugins icons cutefish-core \
    cutefish-dm filemanager terminal xdg-desktop-portal-cutefish settings \
    system-monitor text-editor video-player calendar calculator preview \
    cutefish-compositor wallpapers; do
    [ -d "$repo" ] || continue
    cmake -S "$repo" -B "build/$repo" -DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_BUILD_TYPE=Release \
        -DBUILD_TESTING=OFF || break
    cmake --build "build/$repo" --parallel "$(nproc)" || break
    sudo cmake --install "build/$repo" || break
done
```

The loop stops at the first failure; fix it, usually a missing package, and
run it again. cutefish-toolkit takes the longest, since it builds Skia.

cutefish-computer-use is built with Cargo instead:

```shell
cd ~/cutefish/cutefish-computer-use
cargo build --release
sudo install target/release/cutefish-computer-use /usr/bin/
```

To let an agent use it, see
[Letting agents use the desktop](https://cutefish.org/blog/letting-agents-use-the-desktop).

To build the compositor without X11 application support, which also drops
the `libxcb-*` build dependencies, configure it with
`-DCUTEFISH_ENABLE_XWAYLAND=OFF`.

## 4. Install the runtime dependencies

```shell
sudo apt install accountsservice polkitd gsettings-desktop-schemas xdg-user-dirs locales \
    network-manager bluez upower power-profiles-daemon udisks2 fuse3 \
    pipewire pipewire-pulse wireplumber gnome-keyring libpam-gnome-keyring \
    xdg-desktop-portal xwayland adwaita-icon-theme
```

`xwayland` runs X11 applications; `adwaita-icon-theme` provides the fallback
cursors; `locales` lets Cutefish generate the locale of a language picked in
Settings, so its dates and numbers follow it too. `systemd-coredump` is recommended, so the crash dialog can show
what crashed.

## 5. Log in

### With your existing display manager

Log out, choose **Cutefish (Wayland)** in the session menu of SDDM, GDM or
LightDM, and log in. Nothing else is needed; cutefish-dm can stay
uninstalled.

### With cutefish-dm (optional)

cutefish-dm runs its login screen as a system user, which a source install
has to create once:

```shell
sudo useradd --system --user-group --home-dir /var/lib/cutefish-greeter \
    --create-home cutefish-greeter
```

Then replace the current display manager. This takes effect at the next
boot; replace `sddm` with `gdm3` or `lightdm` as appropriate.

```shell
sudo systemctl disable sddm
sudo systemctl enable cutefish-dm
```

To go back, run the same commands with the names swapped.

## Updating

Pull every repository, fetch Skia and PDFium again in case their pinned
revisions changed, and run the loop in step 3 again, along with the Cargo build of
cutefish-computer-use if you have it:

```shell
cd ~/cutefish
for repo in */; do [ -d "$repo/.git" ] && git -C "$repo" pull --ff-only; done
cutefish-toolkit/scripts/fetch-skia.sh
cutefish-framework/scripts/fetch-pdfium.sh
```

libghostty-vt needs rebuilding only when terminal's README asks for a newer
version.

Log out and back in afterwards. The file manager also draws the desktop, so
a new filemanager build takes effect at the next login.

## Uninstalling

Each build directory records what it installed:

```shell
cd ~/cutefish
for manifest in build/*/install_manifest.txt; do
    xargs -d '\n' sudo rm -f -- < "$manifest"
done
sudo rm -f /usr/bin/cutefish-computer-use
sudo rm -rf /usr/lib/cutefish-ghostty-vt /usr/include/cutefish-ghostty-vt \
    /usr/share/pkgconfig/cutefish-ghostty-vt.pc
```

Switch back to your previous display manager first if you enabled
cutefish-dm.

## Other distributions

Build dependencies are looked up through pkg-config, apart from a few
libraries found by name, noted in the table. Install the package that
provides each module below; on Fedora, `sudo dnf install
'pkgconfig(<module>)'` does this directly, and on Arch Linux,
`pacman -F <module>.pc` names the package.

| Components | pkg-config modules |
|---|---|
| All | `wayland-client` (>= 1.23), `wayland-server`, `wayland-egl`, `wayland-scanner`, `wayland-protocols` (>= 1.39), `egl`, `glesv2`, `xkbcommon`, `libsystemd`, `fontconfig`, `freetype2`, `harfbuzz`, `icu-uc`, `icu-i18n`, `expat`, `libpng`, `libjpeg`, `zlib`, `libwebp`, `libtiff-4`, `libheif`, `libjxl`, `libjxl_threads` |
| cutefish-framework | `libpipewire-0.3` (>= 0.3.65), `libspa-0.2`, `harfbuzz-subset`, `lcms2`, `libopenjp2`, `libhwy`, `absl_flat_hash_set`, `absl_inlined_vector`, `absl_cleanup` |
| cutefish-core | `gbm`, `libdrm`, `libavcodec`, `libavformat`, `libavutil`, `libswscale`, `libva` |
| cutefish-compositor | `gbm`, `libdrm`, `libinput`, `libseat`, `libudev`, `xcb`, `xcb-composite`, `xcb-xfixes`, `xcb-shape`, `xcb-sync`; the PAM library |
| cutefish-dm | the PAM library |
| filemanager | `libarchive`, `fuse3`, `libmtp`, `smbclient`, `libssh`, `libudev`, and the libzip headers and library |
| settings | `xcursor`; the libcrypt library |
| terminal | `libpcre2-8`; `cutefish-ghostty-vt`, built in step 3 |
| text-editor | `libpcre2-8` |
| video-player | `mpv` (>= 2.0, from libmpv 0.35) |
| calendar | `libcurl`, `libical` |
| qt-plugins | Qt 6 Core, Gui, Widgets, DBus, Quick and their private headers; `librsvg-2.0` |

cutefish-computer-use's `.cargo/config.toml` builds offline against Debian's
packaged crates in `/usr/share/cargo/registry`. Elsewhere, delete that file
and Cargo fetches the same crates from crates.io.

Besides libraries, the build needs `git`, `cmake`, `ninja`, `pkg-config`,
`clang`, `python3`, gettext (for translations), `gtk-update-icon-cache`
(for icons), and `curl` and `minisign` (for libghostty-vt). Use the Zig
release step 3 downloads rather than a distribution's Zig, since
libghostty-vt 1.0 needs Zig 0.17. The runtime services in step 4 go by the same names on most
distributions:
NetworkManager, BlueZ, UPower, power-profiles-daemon, UDisks2, PipeWire with
WirePlumber, AccountsService, polkit, GNOME Keyring, xdg-desktop-portal and
Xwayland.
