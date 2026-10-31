# Maintainer: echohumm
# Based on the AUR vesktop package (edu4rdshl).
#
# Vesktop (Vencord/Vesktop) is only the Electron shell: it downloads Vencord at
# runtime from Vendicated/Vencord. This package builds both from release tags,
# applies the local Vencord patch + user plugin, and installs the resulting dist
# to /usr/lib/vesktop/vencord, which Vesktop uses as its default Vencord location.
#
# Bump pkgver (Vesktop tag) and _vencordver (Vencord tag) with repoupd.
pkgname=vesktop
_pkgname=Vesktop
pkgver=1.6.8
_vencordver=1.15.15
pkgrel=4
pkgdesc="Standalone Electron-based Discord app with a customized Vencord built in"
arch=('x86_64' 'aarch64')
url="https://github.com/Vencord/Vesktop"
license=('GPL-3.0-or-later')
depends=('alsa-lib' 'gtk3' 'nss')
makedepends=('pnpm' 'npm' 'nodejs' 'glib2')
optdepends=(
  'libnotify: Notifications'
  'xdg-utils: Open links, files, etc'
)
source=("$_pkgname-$pkgver.tar.gz::https://github.com/Vencord/Vesktop/archive/v${pkgver}.tar.gz"
        "Vencord-$_vencordver.tar.gz::https://github.com/Vendicated/Vencord/archive/refs/tags/v${_vencordver}.tar.gz"
        "vencord-custom.patch"
        "TimestampSeconds.ts"
        "vesktop.desktop"
        "vesktop.sh")
sha256sums=('91dd4630fb5fff153586056451a9587f4c5fbeb048c6d570a35056460415f22f'
            '6d567dd6f119d72fcfb8eb078a71b9ab71c5bab226d726388eabb8423db8b2ee'
            'e1fdef08589658f00046d69e3288c2a2080c2ff9684dfb9570ae9f3b905b7429'
            '0f410fa1218836fbcd5c9165363a8bfe67fad21c78d000122ba41ce47f2a3f13'
            '98fa8f661b065c2d825e24f0055a40ae01d58d23628ad7ebf6914296209dd43c'
            '506c246328af639d6f6a3e52215c7b34af2a6df11d195de6f57a8bbee750cce9')

prepare() {
  # Vencord: local replyTimestamp changes + the added TimestampSeconds user plugin.
  cd "$srcdir/Vencord-$_vencordver"
  patch -Np1 -E -i "$srcdir/vencord-custom.patch"
  install -Dm644 "$srcdir/TimestampSeconds.ts" src/userplugins/TimestampSeconds/index.ts

  # Vesktop: default to the bundled Vencord instead of downloading one into
  # ~/.config/vesktop/sessionData. A vencordDir set in Developer Settings still wins.
  cd "$srcdir/$_pkgname-$pkgver"
  sed -i 's|join(SESSION_DATA_DIR, "vencordFiles")|"/usr/lib/vesktop/vencord"|' src/main/vencordFilesDir.ts
  grep -q '"/usr/lib/vesktop/vencord"' src/main/vencordFilesDir.ts
}

build() {
  # Vencord's build script shells out to git for these; the tarball has no .git.
  export VENCORD_HASH="v$_vencordver"
  export VENCORD_REMOTE="Vendicated/Vencord"

  # Same flags upstream's release workflow uses, plus updater disabled (pacman updates it).
  cd "$srcdir/Vencord-$_vencordver"
  pnpm install --frozen-lockfile
  pnpm build --standalone --disable-updater

  cd "$srcdir/$_pkgname-$pkgver"
  pnpm install --frozen-lockfile
  pnpm package:dir
}

package() {
  local vencord="$srcdir/Vencord-$_vencordver"

  cd "$srcdir/$_pkgname-$pkgver"

  install -d "$pkgdir/usr/lib/$pkgname" "$pkgdir/usr/bin"

  case "${CARCH}" in
    aarch64) folder="linux-arm64-unpacked" ;;
    *)       folder="linux-unpacked" ;;
  esac
  cp -R "dist/${folder}/." "$pkgdir/usr/lib/$pkgname"

  # Bundled Vencord (the four files Vesktop loads, plus the package.json it checks for)
  local f
  for f in vencordDesktopMain.js vencordDesktopPreload.js \
           vencordDesktopRenderer.js vencordDesktopRenderer.css; do
    install -Dm644 "$vencord/dist/$f" "$pkgdir/usr/lib/$pkgname/vencord/$f"
  done
  echo '{}' > "$pkgdir/usr/lib/$pkgname/vencord/package.json"

  install -Dm644 "$srcdir/vesktop.desktop" "$pkgdir/usr/share/applications/vesktop.desktop"
  install -Dm644 LICENSE "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
  install -Dm644 "$vencord/LICENSE" "$pkgdir/usr/share/licenses/$pkgname/LICENSE.vencord"
  install -Dm644 build/icon.svg "$pkgdir/usr/share/icons/hicolor/scalable/apps/$pkgname.svg"
  install -Dm755 "$srcdir/vesktop.sh" "$pkgdir/usr/bin/$pkgname"
}
