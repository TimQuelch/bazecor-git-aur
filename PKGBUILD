# Maintainer: Tim Quelch <tim@tquelch.com>

_pkgname="bazecor"
_branch="development"
pkgname="${_pkgname}-git"
pkgver=1.3.10_rc.3.3368
pkgrel=1
pkgdesc="Graphical configurator for Dygma Raise. Development branch"
url="https://github.com/Dygmalab/Bazecor"
license=("GPL3")
depends=("fuse")
makedepends=("yarn" "git" "nvm" "squashfs-tools" "python")
provides=("${_pkgname}")
conflicts=("${_pkgname}")
arch=("x86_64")
source=("${pkgname}::git+https://github.com/Dygmalab/Bazecor.git#branch=${_branch}")
cksums=("SKIP")
options=(!strip)

# From https://wiki.archlinux.org/title/Node.js_package_guidelines
_ensure_local_nvm() {
    # let's be sure we are starting clean
    which nvm >/dev/null 2>&1 && nvm deactivate && nvm unload
    export NVM_DIR="${srcdir}/.nvm"

    # The init script returns 3 if version specified
    # in ./.nvrc is not (yet) installed in $NVM_DIR
    # but nvm itself still gets loaded ok
    source /usr/share/nvm/init-nvm.sh || [[ $? != 1 ]]
}

pkgver() {
    cd "$srcdir/$pkgname" || return
    printf "%s.%s" \
        "$(grep "\"version\":" package.json | sed 's/^.*"\S*".*"\(\S*\)".*$/\1/' | sed 's/-/_/g')" \
        "$(git rev-list --count HEAD)"
}

prepare() {
    _ensure_local_nvm
    nvm install 18
    nvm use 18
}

build() {
    _ensure_local_nvm
    cd "$srcdir/$pkgname" || return
    yarn || /bin/true           # yarn errors, but seems to still work

    # mksquashfs will throw a error if both the SOURCE_DATE_EPOCH 
    # environment variable and the -all-time flag are set.
    unset SOURCE_DATE_EPOCH

    yarn run make-lin

    _appimage=$(find . -iname "*.AppImage")
    chmod +x "${_appimage}"
    cd .. || return
    ${srcdir}/${pkgname}/"${_appimage}" --appimage-extract

    sed -i -E "s|Exec=AppRun|Exec=/usr/bin/${_pkgname}|" "squashfs-root/${_pkgname^}.desktop"
    chmod -R a-x+rX squashfs-root/usr
}

package() {
    # AppImage
    _appimage=$(find ${srcdir}/${pkgname} -iname "*.AppImage")
    install -Dm755 "${_appimage}" "${pkgdir}/opt/${_pkgname}/${_pkgname}.AppImage"

    # Symlink AppImage
    install -dm755 "${pkgdir}/usr/bin"
    ln -s "/opt/${_pkgname}/${_pkgname}.AppImage" "${pkgdir}/usr/bin/${_pkgname}"

    # Desktop file
    install -Dm644 "${srcdir}/squashfs-root/${_pkgname^}.desktop"\
        "${pkgdir}/usr/share/applications/${_pkgname}.desktop"

    # Icons
    install -dm755 "${pkgdir}/usr/share/"
    cp -a "${srcdir}/squashfs-root/usr/share/icons" "${pkgdir}/usr/share/"
}
