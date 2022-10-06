# Maintainer: Philip Müller <philm[at]manjaro[dot]org>
# Maintainer: Bernhard Landauer <bernhard[at]manjaro[dot]org>
# Maintainer: Mark Wagie <mark at manjaro dot org>
# Contributor: Helmut Stult

# Arch credits:
# Dan McGee <dan@archlinux.org>
# Dave Reisner <dreisner@archlinux.org>

pkgname=pacman
pkgver=6.0.2
_pkgver=1.7.1
pkgrel=2
pkgdesc="A library-based package manager with dependency support"
arch=('x86_64')
url="http://www.archlinux.org/pacman/"
license=('GPL')
groups=('base-devel')
depends=('bash' 'glibc' 'libarchive' 'curl' 'gpgme'
         'gettext' 'gawk' 'coreutils' 'gnupg' 'grep'
         'fakeroot' 'perl' # pacman-contrib deps
         'pacman-mirrors>=4.1.0')
makedepends=('meson' 'asciidoc' 'doxygen')
checkdepends=('python' 'fakechroot')
optdepends=('perl-locale-gettext: translation support in makepkg-template'
            'diffutils: for pacdiff'
            'findutils: for pacdiff --find'
            'mlocate: for pacdiff --locate'
            'sudo: privilege elevation for several scripts'
            'vim: default merge program for pacdiff'
            'haveged: for pacman-init.service')
provides=('pacman-contrib' 'pacman-init' 'libalpm.so')
conflicts=('pacman-contrib' 'pacman-init')
replaces=('pacman-contrib' 'pacman-init')
backup=(etc/pacman.conf
        etc/makepkg.conf)
install=pacman.install
validpgpkeys=('6645B0A8C7005E78DB1D7864F99FFE0FEAE999BD' # Allan McRae <allan@archlinux.org>
              'B8151B117037781095514CA7BBDFFC92306B1121' # Andrew Gregory (pacman) <andrew@archlinux.org>
              '5134EF9EAF65F95B6BB1608E50FB9B273A9D0BB5' # Johannes Löthberg <johannes@kyriasis.com>
              '04DC3FB1445FECA813C27EFAEA4F7B321A906AD9') # Daniel M. Capella <polyzen@archlinux.org>

source=(https://sources.archlinux.org/other/pacman/$pkgname-$pkgver.tar.xz{,.sig}
        https://gitlab.archlinux.org/pacman/pacman-contrib/-/archive/v$_pkgver/pacman-contrib-v$_pkgver.tar.gz
        pacman.conf
        makepkg.conf
        pacman-sync-first-option.patch
        etc-pacman.d-gnupg.mount
        pacman-init.service)
sha256sums=('7d8e3e8c5121aec0965df71f59bedf46052c6cf14f96365c4411ec3de0a4c1a5'
            'SKIP'
            '81ad0af095fa2a686975bc11b4eb3b6602da60196e82819fb7a92f6fae5bf16d'
            'a71fabbf3cce40c8df7b2d9897d01c77bcc51c692258224acf492a4440f0feb7'
            '40c8c0f874a3ce1e2290be95f79b772e1e3661b7e99608e1742349eb192c0f5a'
            '8167155d3a3e15fc4a1b1e989fdb826779e7b3690a52e2ca9d307ae0b1550e1d'
            'b6d14727ec465bb66d0a0358163b1bbfafcb4eaed55a0f57c30aabafae7eed68'
            '65d8bdccdcccb64ae05160b5d1e7f3e45e1887baf89dda36c1bd44c62442f91b')

prepare() {
  cd $pkgname-$pkgver

  # Manjaro patches
  patch -Np1 < "$srcdir"/pacman-sync-first-option.patch

  cd $srcdir/pacman-contrib-v$_pkgver
  ./autogen.sh
}

build() {
  cd "$pkgname-$pkgver"

  meson --prefix=/usr \
        --buildtype=plain \
        -Ddoc=enabled \
        -Ddoxygen=enabled \
        -Dscriptlet-shell=/usr/bin/bash \
        -Dldconfig=/usr/bin/ldconfig \
        build

  meson compile -C build

  cd $srcdir/pacman-contrib-v$_pkgver

  ./configure \
    --prefix=/usr \
    --sysconfdir=/etc \
    --localstatedir=/var
  make
}

check() {
  cd "$pkgname-$pkgver"
  meson test -C build

  make -C ../pacman-contrib-v$_pkgver check
}

package() {
  cd "$pkgname-$pkgver"

  DESTDIR="$pkgdir" meson install -C build

  # install Arch specific stuff
  install -dm755 "$pkgdir/etc"
  install -m644 "$srcdir/pacman.conf" "$pkgdir/etc"
  install -m644 "$srcdir/makepkg.conf" "$pkgdir/etc"

  # install pacman-init
  install -dm755 $pkgdir/usr/lib/systemd/system/
  install -m644 $srcdir/etc-pacman.d-gnupg.mount $pkgdir/usr/lib/systemd/system/etc-pacman.d-gnupg.mount
  install -m644 $srcdir/pacman-init.service $pkgdir/usr/lib/systemd/system/pacman-init.service

  cd $srcdir/pacman-contrib-v$_pkgver

  make DESTDIR="$pkgdir" install

  # replace rankmirrors
  rm "$pkgdir/usr/bin/rankmirrors"
  ln -sfv "/usr/bin/pacman-mirrors" "$pkgdir/usr/bin/rankmirrors"
}
