# Maintainer: WMDE <https://wmde.fun>
# Contributor: System76 <info@system76.com> (original cosmic-settings-daemon)
#
# Builds our fork Lin-WMDE/wmde-settings-daemon (branch wmde). WMDE settings
# backend daemon: brightness/audio/theme/xkb/geoclue integration + varlink audio
# server. Installs alongside cosmic-settings-daemon: own binary (wmde-settings-daemon),
# own D-Bus/config id (fun.wmde.SettingsDaemon) and own varlink socket
# (fun.wmde.Settings), so NO conflicts/replaces/provides of cosmic-settings-daemon
# and NO compat symlinks. Workspace member library crates keep their upstream
# names/dirs (audio-*, config, cosmic-settings-daemon-config, geonames,
# cosmic-pipewire, varlink-server) - they are consumed only in-tree here.
pkgname=wmde-settings-daemon
pkgver=0.1.0
pkgrel=2
pkgdesc="WMDE settings daemon (fork of cosmic-settings-daemon) - owns fun.wmde.SettingsDaemon"
arch=('x86_64')
url="https://wmde.fun"
license=('GPL-3.0-or-later')
# runtime: pipewire (audio backend), libpulse (pactl helper), udev/systemd-libs
# (backlight + logind), libxkbcommon (xkb layout switching), dbus, polkit rules,
# ddcutil (external monitor brightness via ddc-hi). Verify with namcap after build.
depends=('glibc' 'gcc-libs' 'pipewire' 'libpulse' 'libxkbcommon'
         'systemd-libs' 'dbus' 'ddcutil')
makedepends=('rust' 'cargo' 'git' 'just' 'clang' 'lld' 'pkgconf'
             'pipewire' 'libxkbcommon' 'systemd')
# !lto: makepkg's LTO option injects -flto into CFLAGS, which the cc crate uses
# to build libspa-sys's C shim (static_fns.c) as a GCC LTO bitcode-only object.
# ld.lld cannot read that bitcode, so the *_libspa_rs shim wrappers end up as
# undefined symbols at final link. Disabling LTO keeps the shim native + linkable.
options=('!lto')
optdepends=('polkit: privileged locale/keyboard changes'
            'geoclue: automatic timezone and night-light location'
            'wmde-osd: headphone-confirmation and OSD prompts')
source=("$pkgname::git+https://github.com/Lin-WMDE/wmde-settings-daemon.git#branch=wmde")
sha256sums=('SKIP')

pkgver() {
  cd "$srcdir/$pkgname"
  # WMDE version: <COSMIC major.minor>.<WMDE release>.<commits since the nearest tag>.g<short>.
  local desc
  desc=$(git describe --long --tags --abbrev=7 2>/dev/null || true)
  if [ -n "$desc" ]; then
    printf '1.10.2.%s.g%s' "$(printf '%s' "$desc" | sed -E 's/.*-([0-9]+)-g[0-9a-f]+$/\1/')" "$(git rev-parse --short=7 HEAD)"
  else
    printf '1.10.2.%s.g%s' "$(git rev-list --count HEAD)" "$(git rev-parse --short=7 HEAD)"
  fi
}

build() {
  cd "$srcdir/$pkgname"
  # x86-64-v3 (AVX2/BMI2) baseline for the WMDE repo; runs on Haswell+ (and the VM).
  export RUSTFLAGS="${RUSTFLAGS:+$RUSTFLAGS }-C target-cpu=x86-64-v3"
  just build-release --bin wmde-settings-daemon
}

package() {
  cd "$srcdir/$pkgname"
  # installs /usr/bin/wmde-settings-daemon, the system_actions defaults under
  # /usr/share/wmde/fun.wmde.Settings.Shortcuts/v1/ and the polkit rule
  # /usr/share/polkit-1/rules.d/wmde-settings-daemon.rules .
  just rootdir="$pkgdir" prefix=/usr install
  install -Dm644 LICENSE "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}
