# Maintainer: kpcyrd <kpcyrd[at]archlinux[dot]org>
# Contributor: Jean Lucas <jean@4ray.co>

pkgname=signal-desktop
_pkgname=Signal-Desktop
pkgver=1.30.0
pkgrel=1
pkgdesc="Electron application that links with Signal on mobile"
license=('GPL3')
arch=('x86_64')
url="https://signal.org"
depends=('electron' 'libvips')
makedepends=('yarn' 'git' 'nodejs' 'npm' 'python' 'python2')
source=(
  "${pkgname}-${pkgver}.tar.gz::https://github.com/signalapp/${_pkgname}/archive/v${pkgver}.tar.gz"
  "${pkgname}.desktop"
  "openssl-linking.patch"
  "expire-from-source-date-epoch.patch"
  # Cherry-pick a specific commit for the node-spellchecker dependency
  # See https://github.com/atom/node-spellchecker/issues/127
  "https://github.com/atom/node-spellchecker/archive/613ff91dd2d9a5ee0e86be8a3682beecc4e94887.tar.gz"
)
sha512sums=('bc804a1fe3135418cc0b4e8a5db87f86a6f246f97fc0599911816c581cb129239c46f3be036be31cb90e2eea9d64276887a845329e474e443d119071b56606c9'
            'c5ec0bf524e527ecf94207ef6aa1f2671346e115ec15de6d063cde0960151813752a1814e003705fc1a99d4e2eae1b3ca4d03432a50790957186e240527cc361'
            '6b846fdf70dae6c4657de523ec133d2f08325740863660b86e75d032bb07a4b97834ba0eeea4c77000c2c20b11739b8e8deaf06584f9279638e640c4b7633dd5'
            '6673066172d6c367961f3e2d762dd483e51a9f733d52e27d0569b333ad397375fd41d61b8a414b8c9e8dbba560a6c710678b3d105f8d285cb94d70561368d5a2'
            '42f57802fa91dafb6dbfb5a3f613c4c07df65e97f8da84c9a54292c97a4d170f8455461aac8f6f7819d1ffbea4bf6c28488f8950056ba988776d060be3f107dd')
b2sums=('39b94317f440e133d755956e2ebe4705653e13d9c7f0e66c0df676645d90ccd490ca584c14e532fc491a35da436b1b592a5cef819ae0b2715a51fb9947bdd9d4'
        'c0ceb5b903965727714b1848c818877f4b740c4734deafcfaf777046002a445d79cd4d86bbbf3d763dbda8e8d542b60605ae2ca43196ea76b089f2e808926dbc'
        '91fe76cd2ef32bd523aa857a219209f93ca5a6a3f5caa35f67c489a8eb79c8e1e404f453bed9e866e543ed48b9df8e17b45ad2ea8891b48d1502a97589a144af'
        'b8171e6d881a6ffd5588d1cae00ed81412eff1602670003fc1f48b7e6cb2d680340d464b7b38ee8886a8bd8193166ad71e3ad10b0de8b2a397b383b72434e289'
        '53ad39f0bed69c5c5635d2e581f4ce56e3080fee16f184add08adee1164c2d68059d52071478a05cdacf04186f9964250f46f09e80553bf35e1f2f4cd1976d1b')

prepare() {
  cd "${_pkgname}-${pkgver}"

  # Fix SpellChecker build with imminent Node 13
  # See https://github.com/atom/node-spellchecker/issues/127
  sed -r 's#("spellchecker": ").*"#\1file:'"${srcdir}"'/613ff91dd2d9a5ee0e86be8a3682beecc4e94887.tar.gz"#' -i package.json

  # Set system Electron version for ABI compatibility
  sed -r 's#("electron": ").*"#\1'$(cat /usr/lib/electron/version)'"#' -i package.json

  # Allow higher Node versions
  sed 's#"node": "#&>=#' -i package.json

  yarn install

  # Have SQLCipher dynamically link from OpenSSL
  # See https://github.com/signalapp/Signal-Desktop/issues/2634
  patch --forward --strip=1 --input="${srcdir}/openssl-linking.patch"

  # We can't read the release date from git so we use SOURCE_DATE_EPOCH instead
  patch --forward --strip=1 --input="${srcdir}/expire-from-source-date-epoch.patch"
}

build() {
  cd "${_pkgname}-${pkgver}"

  # Gruntfile expects Git commit information which we don't have in a tarball download
  # See https://github.com/signalapp/Signal-Desktop/issues/2376
  yarn generate exec:build-protobuf exec:transpile concat copy:deps sass

  yarn build-release
}

package() {
  cd "${_pkgname}-${pkgver}"

  install -d "${pkgdir}/usr/"{lib,bin}
  cp -a release/linux-unpacked/resources "${pkgdir}/usr/lib/${pkgname}"
  cat << EOF > "${pkgdir}"/usr/bin/${pkgname}
#!/bin/sh

NODE_ENV=production electron /usr/lib/${pkgname}/app.asar "\$@"
EOF
  chmod +x "${pkgdir}/usr/bin/${pkgname}"

  install -Dm 644 "../${pkgname}.desktop" -t "${pkgdir}/usr/share/applications"
  for i in 16 24 32 48 64 128 256 512 1024; do
    install -Dm 644 "build/icons/png/${i}x${i}.png" \
      "${pkgdir}/usr/share/icons/hicolor/${i}x${i}/apps/${pkgname}.png"
  done
}

# vim: ts=2 sw=2 et:
