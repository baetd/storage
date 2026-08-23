pkgname=storage
pkgver=1.5.0
pkgrel=1
pkgdesc="Disk usage and storage information CLI"
arch=('any')
url="https://github.com/baetd/storage"
license=('GPL-3.0-only')

depends=(
    'bash'
    'coreutils'
    'util-linux'
    'gawk'
)

optdepends=(
    'nerd-fonts: Needed for icons'
    'kitty: For better view'
    'sysstat: I/O statistics via iostat'
)

source=("$url/archive/refs/tags/v$pkgver.tar.gz")
sha256sums=('SKIP')

package() {
    install -Dm755 "$pkgname-$pkgver/storage" \
        "$pkgdir/usr/bin/storage"
}
