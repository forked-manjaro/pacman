# Maintainer: Philip Müller <philm[at]manjaro[dot]org>
# Maintainer: Bernhard Landauer <bernhard[at]manjaro[dot]org>
# Maintainer: Mark Wagie <mark at manjaro dot org>
# Contributor: Helmut Stult
# Contributor: Levente Polyak <anthraxx[at]archlinux[dot]org>
# Contributor: Morten Linderud <foxboron@archlinux.org>

pkgname=pacman
pkgver=6.1.0
pkgrel=5
pkgdesc="A library-based package manager with dependency support"
arch=('x86_64')
url="https://www.archlinux.org/pacman/"
license=('GPL-2.0-or-later')
depends=('bash' 'glibc' 'libarchive' 'curl' 'gpgme' 'pacman-mirrorlist'
         'gettext' 'gawk' 'coreutils' 'gnupg' 'grep')
makedepends=('meson' 'asciidoc' 'doxygen')
checkdepends=('python' 'fakechroot')
optdepends=('perl-locale-gettext: translation support in makepkg-template')
provides=('libalpm.so')
backup=(etc/pacman.conf
        etc/makepkg.conf)
options=('strip')
validpgpkeys=('6645B0A8C7005E78DB1D7864F99FFE0FEAE999BD'  # Allan McRae <allan@archlinux.org>
              'B8151B117037781095514CA7BBDFFC92306B1121') # Andrew Gregory (pacman) <andrew@archlinux.org>
source=(https://gitlab.archlinux.org/pacman/pacman/-/releases/v$pkgver/downloads/pacman-$pkgver.tar.xz{,.sig}
        revertme-makepkg-remove-libdepends-and-libprovides.patch::https://gitlab.archlinux.org/pacman/pacman/-/commit/354a300cd26bb1c7e6551473596be5ecced921de.patch
        "$pkgname-fix-msg-unknown-key.patch::https://gitlab.archlinux.org/pacman/pacman/-/commit/6bb95c8856437513ee0ab19226bc090d6fd0fb06.patch"
        "$pkgname-man-gitlab.patch::https://gitlab.archlinux.org/pacman/pacman/-/commit/95f148c2222db608a0d72d5c5577d0c71e7fa199.patch"
        "$pkgname-make-aligned-titles.patch::https://gitlab.archlinux.org/pacman/pacman/-/commit/5e0496260b7d3f9c9fcf2b1c4899e4dbcc20ff03.patch"
        "$pkgname-repo-add-parseopts.patch::https://gitlab.archlinux.org/pacman/pacman/-/commit/0571ee82bff0edbd5ffac2228d4e6ac510b9008e.patch"
        "$pkgname-drop-result-warn.patch::https://gitlab.archlinux.org/pacman/pacman/-/commit/111eed0251238a9d3f90e76d62f2ac01aeccce48.patch"
        "$pkgname-fix-debugedit.patch::https://gitlab.archlinux.org/pacman/pacman/-/commit/bae9594ac1806ce30f2af1de27c49bb101a00d44.patch"
        fetch-signature-and-database-from-same-URL.patch::https://gitlab.archlinux.org/pacman/pacman/-/commit/eb5bf6913835e7553433ef82bdf0a456528f9b50.patch
        pacman.conf
        makepkg.conf
        pacman-sync-first-option.patch)
sha256sums=('5a60ac6e6bf995ba6140c7d038c34448df1f3daa4ae7141d2cad88eeb5f1f9d9'
            'SKIP'
            'b3bce9d662e189e8e49013b818f255d08494a57e13fc264625f852f087d3def2'
            '94c987046c2ff232fa0d395cddc11644840d767806711e04ef34f876a9baf217'
            '0774d7035e34661f74b673d4b0a94be877bdc0158a555b873ec6bd4e2c936377'
            '7bb64910265ce2590f593cdfd302076e49f67a68f8cc792a9aaac572d36fc842'
            '2bbfe40539513ff5775aaf900644c8985ef618f5df9af856b9d571e2501365b0'
            '160515b741aadc876a67f213029f5f62a51ff072ea4aaeb687bbe614035bf72f'
            '1f4e4cc54332e60c9da2bdabf9a80dc11db466535f1a0be298cbf654f0723721'
            '353069264dda744736ac57ef2bb1f7c6617a922252ca8e6be26b08a929cb3410'
            '69dbd2197423fe8be9ec74cb9fa58a819996abc5d19b4569fe84fc9c076397b7'
            '154a1ecd9ac827e19fdd8daa643f60cda4417874e66572238a3fc866753f5eb3'
            '8167155d3a3e15fc4a1b1e989fdb826779e7b3690a52e2ca9d307ae0b1550e1d')

prepare() {
  cd "$pkgname-$pkgver"

  # handle patches
  local -a patches
  patches=($(printf '%s\n' "${source[@]}" | grep '.patch'))
  patches=("${patches[@]%%::*}")
  patches=("${patches[@]##*/}")

  if (( ${#patches[@]} != 0 )); then
    for patch in "${patches[@]}"; do
      if [[ $patch =~ revertme-* ]]; then
        msg2 "Reverting patch $patch..."
        patch -RNp1 < "../$patch"
      else
        msg2 "Applying patch $patch..."
        patch -Np1 < "../$patch"
      fi
    done
  fi
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
}

check() {
  cd "$pkgname-$pkgver"

  meson test -C build
}

package() {
  cd "$pkgname-$pkgver"

  DESTDIR="$pkgdir" meson install -C build

  # install Arch specific stuff
  install -dm755 "$pkgdir/etc"
  install -m644 "$srcdir/pacman.conf" "$pkgdir/etc"
  install -m644 "$srcdir/makepkg.conf" "$pkgdir/etc"

  local wantsdir="$pkgdir/usr/lib/systemd/system/sockets.target.wants"
  install -dm755 "$wantsdir"

  local unit
  for unit in dirmngr gpg-agent gpg-agent-{browser,extra,ssh} keyboxd; do
    ln -s "../${unit}@.socket" "$wantsdir/${unit}@etc-pacman.d-gnupg.socket"
  done
}

# vim: set ts=2 sw=2 et:

