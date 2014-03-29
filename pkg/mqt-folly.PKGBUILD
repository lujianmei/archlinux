# vim:ts=2:sw=2:et
# Maintainer: Joel Teichroeb <joel@teichroeb.net>
# Contributor: Jonas Heinrich <onny@project-insanity.org>
# Contributor: Serge Aleynikov <saleyn@gmail.com>

# If TOOLCHAIN env var is set, then the package name
# will contain "-${toolchain}" suffix in lower case
# otherwise, it'll end with "-gcc"
_toolset=$(tr '[:upper:]' '[:lower:]' <<< ${TOOLCHAIN:-gcc})
TOOLSET=$(tr  '[:lower:]' '[:upper:]' <<< ${_toolset})

_pkgsfx="-${_toolset}"
pkgbase=folly
pkgname=mqt-${pkgbase}${_pkgsfx}
pkgver=657.d9c79af
pkgrel=1
pkgdesc='Folly is an open-source C++ library developed and used at Facebook'
arch=x86_64
url='https://github.com/facebook/folly'
license=(Apache)
depends=(google-glog
         google-gflags
         double-conversion
         mqt-boost${_pkgsfx}
        )
GTEST=gtest-1.6.0
makedepends=(git mqt-boost${_pkgsfx} google-gflags double-conversion python2)
options=(staticlibs buildflags makeflags)
source=(
  git+https://github.com/facebook/folly.git
  http://googletest.googlecode.com/files/${GTEST}.zip
)
# that sucks that the project downloads gtests sources, it should use system libraries
# https://github.com/facebook/folly/issues/48
md5sums=('SKIP'
         '4577b49f2973c90bf9ba69aa8166b786')

install=mqt-${pkgbase}.install

pkgver() {
  cd ${pkgbase}
  v=$(git describe --tags --abbrev=0 2>/dev/null | sed 's/[^0-9\.]//g')
  if [ -n "$v" ]; then
    printf "%s" $v
  else
    printf "%s.%s" "$(git rev-list --count HEAD)" "$(git rev-parse --short HEAD)"
  fi
}

prepare() {
  cd folly/folly
  find -name '*.py' -exec sed -i 's|^#!/usr/bin/env python$|#!/usr/bin/env python2|' {} \;

  cd test
  ln -s "$srcdir"/$GTEST
}

build() {
  local JOBS="$(sed -e 's/.*\(-j *[0-9]\+\).*/\1/' <<< ${MAKEFLAGS})"
  JOBS=${JOBS:- -j$(nproc)}

  echo "==== Building folly ==="

  rm -f "${pkgbase}*.log.*"

  cd "$srcdir"/folly/folly

  grep -q "^CPPFLAGS += -I$GTEST/include -I\.\.$" test/Makefile.am ||
    sed -i "s/^CPPFLAGS += -I$GTEST\/include$/& -I../" test/Makefile.am

  autoreconf --install

  CPPFLAGS="$CPPFLAGS -I/usr/include/double-conversion -Wno-deprecated -g -O3"

  # building shared library fails at it requires libiberty.so, gcc package provides only static library
  ./configure \
    --disable-shared \
    --prefix=/opt/pkg/${pkgbase}/${pkgver} \
    --exec-prefix=/opt/pkg/${pkgbase}/${pkgver}/${TOOLSET} \
    --with-boost=/opt/env/prod/Boost/Current \
    --with-boost-libdir=/opt/env/prod/Boost/Current/${TOOLSET}/lib
  make $JOBS V=0
}

package() {
  cd "${srcdir}"/folly/folly

  echo "==== Packaging folly ==="

  make DESTDIR="${pkgdir}" install

  cd "${pkgdir}"/opt/pkg/${pkgbase}
  ln -vs ${pkgver} current

  # remove gtest libraries installed by the package
  rm "$pkgdir"/opt/pkg/${pkgbase}/${pkgver}/${TOOLSET}/lib/{libgtest,libgtestmain}.a
}