# Maintainer: Maxime Gauduin <alucryd@archlinux.org>
# Contributor: Bartłomiej Piotrowski <bpiotrowski@archlinux.org>
# Contributor: xduugu
# Contributor: Elis Hughes <elishughes@googlemail.com>

pkgname=rtmpdump
pkgver=2.6
pkgrel=2
epoch=1
pkgdesc='A toolkit for RTMP streams'
arch=(x86_64)
url=https://rtmpdump.mplayerhq.hu/
license=(
  GPL-2.0-only
  LGPL-2.1-only
)
depends=(
  glibc
  gnutls
  zlib
)
makedepends=(git)
provides=(librtmp.so)
_commit=138fdb258d9fc26f1843fd1b891180416c9dc575  # master, tagged v2.6
# Fetch only from Seafoam Labs: git.ffmpeg.org advertises fifteen refs at the
# null OID, which aborts a bare clone of the whole repository.
source=("seafoam-rtmpdump::git+https://github.com/Seafoam-Labs/rtmpdump.git#commit=${_commit}"
        nettle-4.patch)
sha256sums=('0cba5d49d41b5e35a34ea8230124ed853eeeac19091d17a4188a47c31a17d352'
            'b9cbbcf14d9fb46962d541544f47226909a7bc7191e2be50cfd0c73c73c4b80d')

pkgver() {
  cd "$srcdir/seafoam-rtmpdump"
  git describe --tags | sed 's/^v//'
}

prepare() {
  cd "$srcdir/seafoam-rtmpdump"
  patch -p1 -i ../nettle-4.patch
}

build() {
  cd "$srcdir/seafoam-rtmpdump"
  sed -e 's/^CRYPTO=OPENSSL/#CRYPTO=OPENSSL/' -e 's/#CRYPTO=GNUTLS/CRYPTO=GNUTLS/' -i Makefile -i librtmp/Makefile
  make \
    OPT="$CFLAGS" \
    XLDFLAGS="$LDFLAGS"
}

package() {
  cd "$srcdir/seafoam-rtmpdump"
  make \
    prefix=/usr \
    sbindir=/usr/bin \
    mandir=/usr/share/man \
    DESTDIR="${pkgdir}" \
    install
}

# vim: ts=2 sw=2 et:
