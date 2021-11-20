# Maintainer: Joakim Soderlund <joakim.soderlund@gmail.com>
# Maintainer: Jan Alexander Steffens (heftig) <heftig@archlinux.org>
# Contributor: Ionut Biru <ibiru@archlinux.org>
# Contributor: Michael Kanis <mkanis_at_gmx_dot_de>

pkgname=mutter-dynamic-buffering
pkgver=41.1
pkgrel=1.3
pkgdesc="A window manager for GNOME (with dynamic triple/double buffering)"
url="https://gitlab.gnome.org/GNOME/mutter"
arch=(x86_64)
license=(GPL)
depends=(dconf gobject-introspection-runtime gsettings-desktop-schemas
         libcanberra startup-notification zenity libsm gnome-desktop upower
         libxkbcommon-x11 gnome-settings-daemon libgudev libinput pipewire
         xorg-xwayland graphene libxkbfile)
makedepends=(gobject-introspection git egl-wayland meson xorg-server
             wayland-protocols)
checkdepends=(xorg-server-xvfb pipewire-media-session python-dbusmock)
provides=(mutter libmutter-9.so)
conflicts=(mutter)
groups=(gnome)
_commit=8de96d3d7c40e6b5289fd707fdd5e6d604f33e8f  # tags/41.1^0
source=("$pkgname::git+https://gitlab.gnome.org/GNOME/mutter.git#commit=$_commit"
        'backports.patch'
        'mr1441.patch'
)

sha256sums=('SKIP'
            'ba41afab8091d066c9201e900f4e9478f9fcb600647aceb08c1b411ccb04028d'
            '592c03f4a492d39d760b174e487b3b2a58e9caef9b9ef886f5aa09abb94b69d3')

pkgver() {
  cd $pkgname
  git describe --tags | sed 's/-/+/g'
}

prepare() {
  cd "$srcdir/$pkgname"
  patch -p1 < "$srcdir/mr1441.patch"
  patch -p1 < "$srcdir/backports.patch"
}

build() {
  CFLAGS="${CFLAGS/-O2/-O3} -fno-semantic-interposition"
  LDFLAGS+=" -Wl,-Bsymbolic-functions"
  arch-meson $pkgname build \
    -D egl_device=true \
    -D wayland_eglstream=true \
    -D installed_tests=false \
    -D profiler=false \
    -D tests=false
  meson compile -C build
}

_check_internal() (
  mkdir -p -m 700 "${XDG_RUNTIME_DIR:=$PWD/runtime-dir}"
  glib-compile-schemas "${GSETTINGS_SCHEMA_DIR:=$PWD/build/data}"
  export XDG_RUNTIME_DIR GSETTINGS_SCHEMA_DIR

  pipewire &
  _p1=$!

  pipewire-media-session &
  _p2=$!

  trap "kill $_p1 $_p2; wait" EXIT

  meson test -C build --print-errorlogs
)

_check_disabled() {
  dbus-run-session xvfb-run -s '-nolisten local' \
    bash -c "$(declare -f _check_internal); _check_internal"
}

package() {
  meson install -C build --destdir "$pkgdir"
}
