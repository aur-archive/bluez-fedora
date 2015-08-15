# $Id$
# Maintainer: Chris Moeller <chris@kode54.net>
# Contributor: Tom Gundersen <teg@jklm.no>
# Contributor: Andrea Scarpino <andrea@archlinux.org>
# Contributor: Geoffroy Carrier <geoffroy@archlinux.org>

pkgname=bluez-fedora
_pkgname=bluez
_pkgver=4.101
pkgver=20130208
pkgrel=1
pkgdesc="Libraries and tools for the Bluetooth protocol stack with patches from the Fedora Project git repository"
url="http://www.bluez.org/"
arch=('i686' 'x86_64')
license=('GPL2')
depends=('dbus-core' 'python2' 'systemd-tools')
makedepends=('gstreamer0.10-base' 'libusb-compat' 'libsndfile' 'libusbx' 'git')
optdepends=("gstreamer0.10-base: bluetooth GStreamer support"
            "alsa-lib: Audio bluetooth devices support"
            "python2-dbus: to run bluez-simple-agent"
            "python2-gobject: to run bluez-simple-agent"
            "libusb-compat: USB adapters support"
            "cups: CUPS backend")
conflicts=('bluez' 'bluez-libs' 'bluez-utils')
provides=('bluez' 'bluez-libs' 'bluez-utils')
replaces=('bluez' 'bluez-libs' 'bluez-utils')
options=('!libtool')
backup=(etc/bluetooth/{main,rfcomm,audio,network,input,serial}.conf
        'etc/conf.d/bluetooth' 'etc/dbus-1/system.d/bluetooth.conf')
source=("http://www.kernel.org/pub/linux/bluetooth/${_pkgname}-${_pkgver}.tar.bz2"
        'bluetooth.conf.d'
	'rc.bluetooth')
_gitroot=http://pkgs.fedoraproject.org/cgit/bluez.git
_gitname=fedora-patches
_gitrev=7d057878581f88788db3c4996b788c4e0a5d7b54
_patches=('bluez-socket-mobile-cf-connection-kit' '0001-Add-sixaxis-cable-pairing-plugin' '0001-input-Add-helper-function-to-request-disconnect' '0002-fakehid-Disconnect-from-PS3-remote-after-10-mins' '0003-fakehid-Use-the-same-constant-as-declared')

build() {
  msg "Retrieving patches from GIT server..."

  if [ -d "${srcdir}/${_gitname}" ] ; then
    cd "${srcdir}/${_gitname}"
    git pull origin master
    git rebase master
  else
    cd "${srcdir}"
    git clone "${_gitroot}" "${_gitname}"
    cd "${_gitname}"
  fi
  git checkout ${_gitrev}

  cd "${srcdir}/${_pkgname}-${_pkgver}"

  for ((i=0; i < ${#_patches[@]}; i++))
  do
    msg "Applying ${_patches[$i]}..."
    patch -p1 < "${srcdir}/${_gitname}/${_patches[$i]}.patch"
  done

  sed -i 's/AM_CONFIG_HEADER/AC_CONFIG_HEADERS/g' configure.ac

  aclocal
  automake --gnu --add-missing
  autoconf

  ./configure --prefix=/usr \
    --sysconfdir=/etc \
    --localstatedir=/var \
    --libexecdir=/lib \
    --enable-gstreamer \
    --enable-alsa \
    --enable-usb \
    --enable-tools \
    --enable-bccmd \
    --enable-dfutool \
    --enable-hid2hci \
    --enable-hidd \
    --enable-pand \
    --enable-dund \
    --enable-cups \
    --enable-wiimote \
    --disable-test \
    --with-systemdunitdir=/usr/lib/systemd/system

  make
}

package() {
  cd ${srcdir}/${_pkgname}-${_pkgver}
  make DESTDIR=${pkgdir} install

  install -Dm755 ${srcdir}/rc.bluetooth ${pkgdir}/etc/rc.d/bluetooth
  
  install -d ${pkgdir}/etc/bluetooth
  install -m644 network/network.conf \
                input/input.conf \
                audio/audio.conf \
                serial/serial.conf \
    ${pkgdir}/etc/bluetooth/
  
  install -Dm644 ${srcdir}/bluetooth.conf.d \
    ${pkgdir}/etc/conf.d/bluetooth

  # FS#27630
  install -Dm755 test/simple-agent "${pkgdir}"/usr/bin/bluez-simple-agent
  install -Dm755 test/test-device "${pkgdir}"/usr/bin/bluez-test-device
  install -Dm755 test/test-input "${pkgdir}"/usr/bin/bluez-test-input
  sed -i 's#/usr/bin/python#/usr/bin/python2#' \
    "${pkgdir}"/usr/bin/bluez-simple-agent \
    "${pkgdir}"/usr/bin/bluez-test-device \
    "${pkgdir}"/usr/bin/bluez-test-input
}
md5sums=('902b390af95c6c5d6d1a17d94c8344ab'
         '7412982b440f29fa7f76a41a87fef985'
         '864cbd24e6efc3592e9284b0b5fb2cfd')
