# vim:ts=2:sw=2:et
# Contributor: Serge Aleynikov <saleyn@gmail.com>

# If TOOLCHAIN env var is set, then the package name
# will contain "-${toolchain}" suffix in lower case
# otherwise, it'll end with "-gcc"
_toolset=$(tr '[:upper:]' '[:lower:]' <<< ${TOOLCHAIN:-gcc})
TOOLSET=$(tr  '[:lower:]' '[:upper:]' <<< ${_toolset})

_pkgsfx=-${_toolset}
pkgbase=scribe
pkgname=${pkgbase}-git${_pkgsfx}
pkgver=122.4452362
pkgrel=1
pkgdesc='Log data aggregator'
arch=('x86_64')
url='https://github.com/facebook/scribe'
options=('!libtool' buildflags makeflags)
license=('Apache')
depends=(mqt-boost${_pkgsfx} mqt-thrift${_pkgsfx})
makedepends=(git mqt-boost${_pkgsfx} mqt-thrift${_pkgsfx})
source=("git+https://github.com/saleyn/scribe.git#branch=ssl")
md5sums=('SKIP')

install=${pkgbase}.install

pkgver() {
  cd ${pkgbase}
  v=$(git describe --tags --abbrev=0 2>/dev/null | sed 's/[^0-9\.]//g')
  if [ -n "$v" ]; then
    printf "%s" $v
  else
    printf "%s.%s" "$(git rev-list --count HEAD)" "$(git rev-parse --short HEAD)"
  fi
}

build() {
  local JOBS="$(sed -e 's/.*\(-j *[0-9]\+\).*/\1/' <<< ${MAKEFLAGS})"
  JOBS=${JOBS:- -j$(nproc)}

  echo "==== Building ${pkgname} ==="

  rm -f "${pkgbase}*.log.*"

  cd "${srcdir}/${pkgbase}"

  autoreconf -if

  ENVDIR=/opt/env/prod
  BOOST_DIR=${ENVDIR}/Boost/Current
  THRIFT_DIR=${ENVDIR}/Thrift/Current

  ./configure \
    CXXFLAGS="-g -O3" \
    LDFLAGS="-Wl,-rpath,${BOOST_DIR}/${TOOLSET}/lib -Wl,-rpath,${THRIFT_DIR}/${TOOLSET}/lib" \
    BOOST_LIB_DIR="${BOOST_DIR}/${TOOLSET}/lib" \
    --enable-silent-rules \
    --with-boost="${BOOST_DIR}" \
    --with-thriftpath="${THRIFT_DIR}" \
    --with-thriftlibpath="${THRIFT_DIR}/${TOOLSET}/lib" \
    --with-fb303path="${THRIFT_DIR}" \
    --with-fb303libpath="${THRIFT_DIR}/${TOOLSET}/lib"

  #--prefix="/opt/pkg/${pkgbase}/${pkgver}" \
  #--exec-prefix="/opt/pkg/${pkgbase}/${pkgver}/${TOOLSET}" \

  make $JOBS
}

package() {
  cd "${srcdir}"/${pkgbase}

  echo "==== Packaging ${pkgname} ==="

  make DESTDIR="${pkgdir}" install

#  install -Dm644 LICENSE "${pkgdir}"/usr/share/licenses/${pkgname}/LICENSE

#  cd "${pkgdir}"/opt/pkg/${pkgbase}
#  ln -vs ${pkgver} current
}