# Maintainer: Joakim Soderlund <joakim.soderlund@gmail.com>
# Maintainer: Jan Alexander Steffens (heftig) <heftig@archlinux.org>
# Contributor: Ionut Biru <ibiru@archlinux.org>
# Contributor: Michael Kanis <mkanis_at_gmx_dot_de>

pkgname=mutter-dynamic-buffering
pkgver=41.1
pkgrel=1.1
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
        '268e0c1c.patch'
        'a34ad6f0.patch'
        'afbbce8b.patch'
        'mr1441.patch'
)

sha256sums=('SKIP'
            '319f43fb0b1f4ce54d301413deb1016a6adbad8edcc087040182ab5d6b00972f'
            '491bcdac0ea7133ff384e96bae9e1640b725df32ecd9a3ec69d1bf8828675597'
            'f82b35b513331ed03786b88183175e42ff08ed67140f2b58dd043520b5aa7cdd'
            '592c03f4a492d39d760b174e487b3b2a58e9caef9b9ef886f5aa09abb94b69d3')

pkgver() {
  cd $pkgname
  git describe --tags | sed 's/-/+/g'
}

prepare() {
  cd "$srcdir/$pkgname"
  patch -p1 < "$srcdir/mr1441.patch"
  patch -p1 < "$srcdir/268e0c1c.patch"
  patch -p1 < "$srcdir/a34ad6f0.patch"
  patch -p1 < "$srcdir/afbbce8b.patch"
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
