# Maintainer: Thore Bödecker <foxxx0@archlinux.org>
# Contributor: Sébastien "Seblu" Luttringer <seblu@archlinux.org>

pkgbase='ceph'
pkgname=('ceph' 'ceph-libs')
pkgver=13.2.1
pkgrel=3
pkgdesc='Distributed, fault-tolerant storage platform delivering object, block, and file system'
arch=('x86_64')
url='https://ceph.com/'
license=('GPL')
makedepends=('bc' 'boost' 'boost-libs' 'cmake' 'cpio' 'crypto++' 'curl' 'cython'
  'cython2' 'expat' 'fcgi' 'fuse2' 'gcc-libs' 'git' 'glibc' 'gperf' 'gperftools'
  'gptfdisk' 'inetutils' 'jq' 'junit' 'keyutils' 'leveldb' 'libaio'
  'libatomic_ops' 'libedit' 'systemd-libs' 'libutil-linux' 'libxml2' 'lsb-release'
  'lz4' 'ncurses' 'nss' 'parted' 'pcre' 'procps-ng' 'python2-cherrypy'
  'python2-jinja' 'python2-nose' 'python2-pecan' 'python2-pip' 'python2-bcrypt'
  'python2-prettytable' 'python2-pyopenssl' 'python2-setuptools' 'python2-routes'
  'python2-sphinx' 'python2-tox' 'python2-virtualenv' 'python2-werkzeug' 'sed'
  'snappy' 'socat' 'systemd' 'valgrind' 'xfsprogs' 'xmlstarlet' 'yasm'
  'zlib' 'zstd' 'cunit' 'oath-toolkit' 'fontconfig')
options=('emptydirs')
source=("https://download.ceph.com/tarballs/${pkgbase}-${pkgver}.tar.gz"
        'ceph.sysusers'
        'boost-1.67.patch'
        'fix-ceph_disk-python-interpreter.patch'
        'fix-or-disable-broken-tests.patch'
        'fix-python2-paths.patch'
        'remove-distro-version-detection.patch')
sha512sums=('411218ea6037bdf9425a741720ea89876e893e64fbfa518c9584ba581f805fde2c82b13ed47313279665e5d20f36223fc8d5c27055b580c72b22c2004e4da81b'
            '4354001c1abd9a0c385ba7bd529e3638fb6660b6a88d4e49706d4ac21c81b8e829303a20fb5445730bdac18c4865efb10bc809c1cd56d743c12aa9a52e160049'
            'b886c3f2b2a2b32d4033225b5f28cef98ca96e3f4fe2d04cfdbdb3141bbefd81895284abe9c9b75dcad156d54a93dc938bd8d9a45056b4de12855bbbdf0870ca'
            '7abd94a333fb0d6c9f7156d69ed6d4bf123f0f3030407f4347209d677b282e5023664d43e74a21a27b7856d3493ae469a17ea8a810331c7266018cc34eee4841'
            '915bb02b91a2e7c6e21243e5d4d80847aedaf6f899e3bac55f4ccae4bb8386c434aad7f052dd449fd510c103f7796bd0a25c0cfac72ee8d1e94d230cf2f0550a'
            'b8ce48ab190b2408f38a7e78d755d762994d271f7bbf88f51392572d1976c4f81e8fad78eb38051df8ef287d70554214624c7b45359d4bef2c11d9ae7d61151c'
            '02c9e8fd3c23fb4c9c4c576ee6d06e8525ca31decfd964fb7231e73c98fe2987a483dda680969752186f0918f47d9af4fb09a4901e5319077f45d870906716da')

prepare() {
  cd "${srcdir}/${pkgbase}-${pkgver}"
  # apply patch from the source array (should be a pacman feature)
  local filename
  for filename in "${source[@]}"; do
    if [[ "$filename" =~ \.patch$ ]]; then
      msg2 "Applying patch ${filename##*/}"
      patch -p1 -N -i "$srcdir/${filename##*/}"
    fi
  done

  # remove tests that require root privileges
  rm src/test/cli/ceph-authtool/cap*.t

  # remove broken tests
  rm src/test/cli/crushtool/build.t
  rm -rf qa/btrfs
  rm src/btrfs_ioc_test.c

  # this test will try to perform btrfs operations when a btrfs mount
  # is active on the build host, which will fail
  if mount | grep 'type btrfs' &>/dev/null; then
    sed -i '/run-tox-ceph-disk/d' src/test/CMakeLists.txt
  fi

  # fix python interpreter -> python2
  for file in \
    src/ceph-create-keys \
    src/mount.fuse.ceph \
    src/ceph-detect-init/ceph_detect_init/main.py
  do
    sed -i 's|#!/usr/bin/env python|#!/usr/bin/env python2|' "${file}"
  done
}

build() {
  cd "${srcdir}/${pkgbase}-${pkgver}"

  mkdir -p build
  cd build

  # experimental in luminous: (and currently broken with boost 1.66)
  # RADOSGW_BEAST_FRONTEND
  # -> disabled

  cmake \
    -DCMAKE_INSTALL_PREFIX=/usr \
    -DCMAKE_INSTALL_SYSCONFDIR=/etc \
    -DCMAKE_INSTALL_SBINDIR=/usr/bin \
    -DCMAKE_INSTALL_LIBDIR=/usr/lib \
    -DCMAKE_INSTALL_LIBEXECDIR=/usr/lib \
    -DWITH_BABELTRACE=OFF \
    -DWITH_CEPHFS=ON \
    -DWITH_FUSE=ON \
    -DWITH_LTTNG=OFF \
    -DWITH_LZ4=ON \
    -DWITH_MGR=ON \
    -DWITH_NSS=ON \
    -DPYTHON_INCLUDE_DIR=/usr/include/python2.7 \
    -DWITH_RADOSGW=ON \
    -DWITH_RADOSGW_BEAST_FRONTEND=OFF \
    -DWITH_RDMA=OFF \
    -DWITH_SSL=ON \
    -DWITH_SYSTEM_BOOST=ON \
    -DWITH_SYSTEMD=ON \
    -DWITH_TESTS=ON \
    -DWITH_XFS=ON \
    -DENABLE_SHARED=ON \
    -DWITH_MGR_DASHBOARD_FRONTEND=OFF \
    ..

  make all
}

check() {
  cd "${srcdir}/${pkgbase}-${pkgver}/build"

  export CTEST_PARALLEL_LEVEL="$(nproc)"
  make check

  # sometimes processes are not properly terminated...
  for process in ceph-mon ceph-mgr ceph-osd; do
    pkill -9 "$process" || true
  done
}

package_ceph-libs() {
  depends=('boost-libs' 'curl' 'glibc' 'keyutils' 'leveldb' 'libaio'
    'libutil-linux' 'lz4' 'nss' 'python2' 'xfsprogs' 'oath-toolkit')

  cd "${srcdir}/${pkgbase}-${pkgver}/build"

  # main install
  make DESTDIR="$pkgdir" install

  # remove stuff that goes into the ceph package
  rm -rf "${pkgdir}"/usr/lib/{ceph/mgr,systemd,sysusers.d,tmpfiles.d}
  rm -rf "${pkgdir}/usr/share"
  rm -rf "${pkgdir}/usr/sbin"
  rm -rf "${pkgdir}/usr/bin"
  rm -rf "${pkgdir}/etc"
  rm -rf "${pkgdir}/var"
}

package_ceph() {
  depends=('ceph-libs' 'boost-libs' 'curl' 'fuse2' 'glibc' 'gperftools'
    'keyutils' 'leveldb' 'libaio' 'systemd-libs' 'libutil-linux' 'python2-routes'
    'lsb-release' 'ncurses' 'nss' 'python2' 'python2-cherrypy' 'python2-bcrypt'
    'python2-jinja' 'python2-mako' 'python2-pecan' 'python2-prettytable'
    'python2-pyopenssl' 'python2-setuptools' 'python2-singledispatch'
    'python2-webob' 'python2-werkzeug' 'snappy' 'xfsprogs' 'python2-requests')

  cd "${srcdir}/${pkgbase}-${pkgver}/build"

  # main install
  make DESTDIR="$pkgdir" install

  # remove stuff that is in the ceph-libs package
  find "${pkgdir}/usr/lib" -maxdepth 1 -type f -delete
  find "${pkgdir}/usr/lib" -maxdepth 1 -type l -delete
  find "${pkgdir}/usr/lib/ceph" -maxdepth 1 -type f -delete
  find "${pkgdir}/usr/lib/ceph" -maxdepth 1 -type l -delete
  rm -rf "${pkgdir}"/usr/lib/{ceph/{compressor,crypto,erasure-code},python2.7,rados-classes}
  rm -rf "${pkgdir}/usr/include"

  # install tmpfiles.d and sysusers.d stuff
  install -Dm644 "${srcdir}/${pkgbase}-${pkgver}/systemd/ceph.tmpfiles.d" \
    "${pkgdir}/usr/lib/tmpfiles.d/${pkgbase}.conf"
  install -Dm644 "${srcdir}/ceph.sysusers" \
    "${pkgdir}/usr/lib/sysusers.d/${pkgbase}.conf"

  # remove debian init script
  rm -rf "${pkgdir}/etc/init.d"

  # fix sbin dir (cmake opt seems to have no effect)
  mv "${pkgdir}"/usr/sbin/* "${pkgdir}/usr/bin/"
  rm -rf "${pkgdir}/usr/sbin"

  # remove drop.ceph.com ssh stuff
  rm -f "${pkgdir}/usr/share/ceph/{{known_hosts,id_rsa}_drop.ceph.com,.pub}"

  # fix bash completions path
  install -d -m 755 "${pkgdir}/usr/share/bash-completion"
  mv "$pkgdir"/{etc/bash_completion.d,usr/share/bash-completion/completions}

  # fix EnvironmentFile location in systemd service files
  sed -i 's|/etc/sysconfig/|/etc/conf.d/|g' "${pkgdir}"/usr/lib/systemd/system/*.service

  # prepare some paths and set correct permissions
  install -D -d -m750 -o   0 -g 340 "${pkgdir}/etc/ceph"
  install -D -d -m750 -o 340 -g 340 "${pkgdir}/var/log/ceph"
  install -D -d -m750 -o 340 -g 340 "${pkgdir}/var/lib/ceph"
  install -D -d -m750 -o 340 -g 340 "${pkgdir}/var/lib/ceph/bootstrap-mds"
  install -D -d -m750 -o 340 -g 340 "${pkgdir}/var/lib/ceph/bootstrap-osd"
  install -D -d -m750 -o 340 -g 340 "${pkgdir}/var/lib/ceph/bootstrap-rgw"
  install -D -d -m750 -o 340 -g 340 "${pkgdir}/var/lib/ceph/mon"
  install -D -d -m750 -o 340 -g 340 "${pkgdir}/var/lib/ceph/mgr"
  install -D -d -m750 -o 340 -g 340 "${pkgdir}/var/lib/ceph/osd"
}

# vim:set ts=2 sw=2 et:
