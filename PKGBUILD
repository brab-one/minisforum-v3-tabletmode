# Maintainer: brab-one <brab at geldhabi dot net>

pkgname=minisforum-v3-tabletmode
pkgver=1.0.0
pkgrel=1
pkgdesc="Automatic tablet mode and working auto-rotate on the Minisforum V3"
arch=('any')
url='https://github.com/brab-one/minisforum-v3-tabletmode'
license=('MIT')
depends=('python' 'systemd' 'iio-sensor-proxy')
optdepends=('minisforum-v3-dsdt: expose the accelerometer to the kernel'
            'minisforum-v3-accelerometer: correct accelerometer mount matrix'
            'maliit-keyboard: on-screen keyboard in tablet mode')
install="${pkgname}.install"
source=('v3-tabletmode-daemon'
        'v3-tabletmode'
        'v3-tabletmode.service'
        'v3-tabletmode.desktop'
        'v3-tabletmode-uinput.conf'
        '81-iio-sensor-proxy-force-poll.rules'
        'LICENSE')
sha256sums=('68c35c7e20850ded821c48175675bc0a0223579c123975e92b46a194603ec9df'
            '92d2c7927aa77fdef527d5a5c259ca17579fa8652acfa65c77535b6c7824dd31'
            'fe3902df6c70614bb5b9a2a4bbf81f83d690b7e91cc761823f99f21525787006'
            '27fa9e7a46060586068f6b8212ae654e77859437654bb895454aa6a4c890d9bf'
            'a771c9695d7283f7771adc00b680bd27391e6ac00e9fd026f4796067ee9a87eb'
            '878131be558200d932a2e30ae4961908dfcecedeeb8ac6f8c9354557bf89d781'
            '81e6b74bdbdf67fb75054827e474c9447a0b7b0abd438e639f7fe18fb851e3ec')

package() {
    install -Dm755 "$srcdir/v3-tabletmode-daemon" "$pkgdir/usr/bin/v3-tabletmode-daemon"
    install -Dm755 "$srcdir/v3-tabletmode"        "$pkgdir/usr/bin/v3-tabletmode"

    install -Dm644 "$srcdir/v3-tabletmode.service" \
        "$pkgdir/usr/lib/systemd/system/v3-tabletmode.service"
    install -Dm644 "$srcdir/v3-tabletmode-uinput.conf" \
        "$pkgdir/usr/lib/modules-load.d/v3-tabletmode.conf"
    install -Dm644 "$srcdir/v3-tabletmode.desktop" \
        "$pkgdir/usr/share/applications/v3-tabletmode.desktop"

    # force iio-sensor-proxy onto its polling driver; the LSM6DS3TR-C has no
    # IIO trigger, so buffered mode blocks forever and auto-rotate never starts
    install -Dm644 "$srcdir/81-iio-sensor-proxy-force-poll.rules" \
        "$pkgdir/usr/lib/udev/rules.d/81-iio-sensor-proxy-force-poll.rules"

    install -Dm644 "$srcdir/LICENSE" "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}
