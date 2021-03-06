# vim:ts=2:sw=2:et
# Maintainer: Joel Teichroeb <joel@teichroeb.net>
# Contributor: Jonas Heinrich <onny@project-insanity.org>
# Contributor: Serge Aleynikov <saleyn@gmail.com>

# If TOOLCHAIN env var is set, then the package name
# will contain "-${toolchain}" suffix in lower case
# otherwise, it'll end with "-gcc"
TOOLSET=$(tr '[:upper:]' '[:lower:]' <<< ${TOOLCHAIN:-gcc})

pkgbase=erl-libs
pkgname=mqt-${pkgbase}
pkgver=1.1
pkgrel=1
pkgdesc='Collection of open-source Erlang libraries'
arch=x86_64
license=(Apache)
GTEST=gtest-1.6.0
makedepends=(git rebar saxon-he)
source=(
  git+https://github.com/erlang/pmod_transform
  git+https://github.com/saleyn/erlsom.git
  git+https://github.com/saleyn/erlcfg.git
  git+https://github.com/saleyn/proto.git
  emmap::git+https://github.com/saleyn/emmap.git#branch=atomic
  git+https://github.com/saleyn/secdb.git
  git+https://github.com/erlware/rebar_vsn_plugin.git
  git+https://github.com/erlware/erlcron.git
  git+https://github.com/saleyn/erlfix.git
  git+https://github.com/esl/edown.git
  git+https://github.com/esl/parse_trans.git
  git+https://github.com/extend/sheriff.git
  mysql::git+https://github.com/mysql-otp/mysql-otp
  #emysql::git+https://github.com/Eonblast/Emysql.git
  cowlib::git+https://github.com/ninenines/cowlib.git#branch=master
  git+https://github.com/ninenines/ranch.git
  git+https://github.com/ninenines/cowboy.git
  git+https://github.com/saleyn/erlexec
  git+https://github.com/saleyn/util
  git+https://github.com/saleyn/gen_timed_server
  git+https://github.com/uwiger/gproc.git
  git+https://github.com/basho/lager.git
  git+https://github.com/DeadZen/goldrush.git
  git+https://github.com/mochi/mochiweb.git
  git+https://github.com/davisp/jiffy.git
  git+https://github.com/manopapad/proper.git
  git+https://github.com/saleyn/getopt.git#branch=format_error
)

md5sums=( $(for i in `seq 1 ${#source[@]}`; do echo 'SKIP'; done) )

install=mqt-${pkgbase}.install

prepare() {
  rm util/src/decompiler.erl
  cd $srcdir
  mkdir -p "lager/deps"
  cd $srcdir/lager/deps
  LIB=goldrush; rm -f $LIB; ln -s ../../$LIB

  cd $srcdir
  mkdir -p jiffy/deps
  cd $srcdir/jiffy/deps
  LIB=proper; rm -f $LIB; ln -s ../../$LIB

  cd $srcdir
  mkdir -p parse_trans/deps
  cd $srcdir/parse_trans/deps
  LIB=edown; rm -f $LIB; ln -s ../../$LIB

  cd $srcdir
  mkdir -p sheriff/deps
  cd $srcdir/sheriff/deps
  LIB=parse_trans; rm -f $LIB; ln -s ../../$LIB
  LIB=edown; rm -f $LIB; ln -s ../../$LIB

  cd $srcdir
  mkdir -p erlcron/deps
  cd $srcdir/erlcron/deps
  LIB=rebar_vsn_plugin; rm -f $LIB; ln -s ../../$LIB

  cd $srcdir
  mkdir -p emmap/deps
  cd $srcdir/emmap/deps
  LIB=edown; rm -f $LIB; ln -s ../../$LIB

  cd $srcdir
  mkdir -p erlcfg/deps
  cd $srcdir/erlcfg/deps
  LIB=pmod_transform; rm -f $LIB; ln -s ../../$LIB

  cd $srcdir
  mkdir -p cowboy/deps
  cd $srcdir/cowboy/deps
  LIB=cowlib; rm -f $LIB; ln -s ../../$LIB
  LIB=ranch;  rm -f $LIB; ln -s ../../$LIB

  unset LIB
}

build() {
  local JOBS="$(sed -e 's/.*\(-j *[0-9]\+\).*/\1/' <<< ${MAKEFLAGS})"
  JOBS=${JOBS:- -j$(nproc)}

  rm -f ../${pkgbase}*.log.*

  for d in $(find ${srcdir} -maxdepth 1 -type d -not -name src -not -name pkg -printf "%f\n")
  do
    case $d in
      proper)           cd ${srcdir}/$d && make fast;;
      rebar_vsn_plugin) cd ${srcdir}/$d && make compile;;
      stockdb)          cd ${srcdir}/$d && make app;;
      erlcron)          cd ${srcdir}/$d && make compile;;
      emmap)            cd ${srcdir}/$d && rebar compile;;
      *)                cd ${srcdir}/$d
                        echo "Making $d ($PWD})"
                        make $JOBS;;
    esac
  done
}

inst_dir() {
  local base="${pkgdir}/opt/pkg/${pkgbase}/${pkgver}/$1"
  echo "${PWD}" 1>&2
  local ver=$(sed -n 's/.*{vsn,[[:space:]]*"[^0-9\.]*\([^"-+]\+\)\([-+][^"]\+\)\{0,1\}"}.*$/\1/p' ebin/$1.app 2>/dev/null || true)
  if [ -n "$ver" ]; then
    printf "${base}-${ver}"
  else
    ver=$(git describe --tags --abbrev=0 2>/dev/null | sed 's/^[^0-9\.]*//')
    if [ -n "$ver" ]; then
      printf "${base}-${ver}"
    else
      printf "${base}-0.1.%s" $(git rev-list --count HEAD) #$(git rev-parse --short HEAD)
    fi
  fi
}

package() {

  echo "==== Packaging ${pkgbase} ==="
  BASE="${pkgdir}/opt/pkg/${pkgbase}/${pkgver}"

  cd "${srcdir}"

  for d in $(ls -dtr */)
  do
    d=${d%/}
    cd "${srcdir}/${d}"
    d=${d##*/}
    DIR=$(inst_dir $d)
    mkdir -vp $DIR
    INC=""
    [ -d "bin"     ] && INC+=" $(find bin     -type f -maxdepth 1)"
    [ -d "ebin"    ] && INC+=" $(find ebin    -type f \( -name '*.app' -o -name '*.beam' \) -maxdepth 1)"
    [ -d "src"     ] && INC+=" $(find src     -type f -maxdepth 1)"
    [ -d "include" ] && INC+=" $(find include -type f -name '*.hrl' -maxdepth 1)"
    [ -d "test"    ] && INC+=" $(find test    -type f -name '*.erl' -maxdepth 1)"
    [ -d "priv"    ] && INC+=" $(find priv    -type f -maxdepth 1)"
    for i in $INC; do
      install -m $(if [ -x "$i" ]; then echo 755; else echo 644; fi) -D $i $DIR/$i;
    done
    #if [ -d "deps" ]; then
    #  cd deps
    #  for i in */ebin/*.{app,beam} */src/*.erl; do install -v -m 644 -D $i $DIR/deps/$i; done
    #fi
  done
  
  cd "${srcdir}"/erlexec
  DIR=$(inst_dir erlexec)
  for i in priv/*/*; do install -m 755 -D $i $DIR/$i; done

  cd "${srcdir}"/mochiweb
  DIR=$(inst_dir mochiweb)
  for i in examples/*/*; do install -m 644 -D $i $DIR/$i; done

  cd "${pkgdir}"/opt/pkg/${pkgbase}
  ln -vs ${pkgver} current
}
