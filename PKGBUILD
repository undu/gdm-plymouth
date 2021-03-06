# Maintainer: Sebastian Lau <lauseb644 _at_ gmail _dot_ com>
# Contributor: Jan Alexander Steffens (heftig) <jan.steffens@gmail.com>
# Contributor: Jan de Groot <jgc@archlinux.org>
# Contributor: Damian01w <damian01w@gmail.com>

_pkgbase=gdm-gnome
_pkgver=3-28
_pkghash=b3224275a137b786b1dd4e5fa93024deb57d41e0
pkgbase=gdm-plymouth
pkgname=(gdm-plymouth libgdm-plymouth)
pkgver=3.28.1
pkgrel=2
pkgdesc="Gnome Display Manager with Plymouth support."
arch=('x86_64')
license=(GPL)
url="http://www.gnome.org"
depends=('plymouth' 'gnome-shell>=3.28.1' 'gnome-session' 'upower' 'xorg-xrdb' 'xorg-server' 'xorg-server-xwayland' 'xorg-xhost')
makedepends=('intltool' 'yelp-tools' 'gobject-introspection')
checkdepends=('check')
source=("https://gitlab.gnome.org/GNOME/gdm/-/archive/gnome-3-28/gdm-gnome-3-28-1.tar.bz2"
        "0002-Xsession-Don-t-start-ssh-agent-by-default.patch"
        "gdm.sysusers")
sha256sums=('778d3d177af4db0713ae0852901ababf69ffce85d62a1996f0501cd6bea26365'
            '63f99db7623f078e390bf755350e5793db8b2c4e06622caf42eddc63cd39ecca'
            '6d9c8e38c7de85b6ec75e488585b8c451f5d9b4fabd2a42921dc3bfcc4aa3e13')

prepare() {
  cd ${_pkgbase}-${_pkgver}-${_pkghash}

  patch -Np1 -i ../0002-Xsession-Don-t-start-ssh-agent-by-default.patch

  NOCONFIGURE=1 ./autogen.sh
}

build() {
  cd ${_pkgbase}-${_pkgver}-${_pkghash}
  ./configure \
    --prefix=/usr \
    --sbindir=/usr/bin \
    --sysconfdir=/etc \
    --libexecdir=/usr/lib \
    --localstatedir=/var \
    --disable-static \
    --disable-schemas-compile \
    --enable-gdm-xsession \
    --enable-ipv6 \
    --with-plymouth \
    --with-default-pam-config=arch \
    --with-default-path=/usr/local/bin:/usr/local/sbin:/usr/bin \
    --without-tcp-wrappers

  sed -i -e 's/ -shared / -Wl,-O1,--as-needed\0/g' libtool

  make
}

check() {
  cd ${_pkgbase}-${_pkgver}-${_pkghash}
  make check
}

package_gdm-plymouth() {
  depends+=(libgdm-plymouth)
  provides=("gdm")
  conflicts=("gdm")
  optdepends=('fprintd: fingerprint authentication')
  backup=(etc/pam.d/gdm-autologin etc/pam.d/gdm-fingerprint etc/pam.d/gdm-launch-environment
          etc/pam.d/gdm-password etc/pam.d/gdm-smartcard etc/gdm/custom.conf
          etc/gdm/Xsession etc/gdm/PostSession/Default etc/gdm/PreSession/Default)
  groups=(gnome)
  install=gdm-plymouth.install

  cd ${_pkgbase}-${_pkgver}-${_pkghash}
  make DESTDIR="$pkgdir" install

  chown -R 120:120 "$pkgdir/var/lib/gdm"

  # Unused or created at start
  rm -r "$pkgdir"/var/{cache,log,run}

  install -Dm644 ../gdm.sysusers "$pkgdir/usr/lib/sysusers.d/gdm.conf"

  ### Split libgdm
  make -C libgdm DESTDIR="$pkgdir" uninstall
  mv "$pkgdir/usr/share/glib-2.0/schemas/org.gnome.login-screen.gschema.xml" "$srcdir"
}

package_libgdm-plymouth() {
  pkgdesc="GDM support library including Plymouth support"
  depends=(systemd glib2 dconf)
  provides=("libgdm")
  conflicts=("libgdm")

  cd ${_pkgbase}-${_pkgver}-${_pkghash}
  make -C libgdm DESTDIR="$pkgdir" install
  install -Dm644 "$srcdir/org.gnome.login-screen.gschema.xml" \
    "$pkgdir/usr/share/glib-2.0/schemas/org.gnome.login-screen.gschema.xml"
}
