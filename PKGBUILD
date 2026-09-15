# Maintainer: HurricanePootis <hurricanepootis@protonmail.com>

pkgname=clvk-git
pkgver=r817.f07937a
pkgrel=1
pkgdesc='Experimental implementation of OpenCL 3.0 on Vulkan'
arch=('aarch64')
url='https://github.com/kpet/clvk'
license=('Apache-2.0')
depends=('vulkan-icd-loader' 'zlib' 'zstd' 'spirv-tools' 'libstdc++' 'glibc' 'libgcc')
makedepends=('gcc' 'git' 'cmake' 'python' 'opencl-headers' 'spirv-headers'
             'spirv-tools' 'spirv-llvm-translator' 'vulkan-headers' 'ninja')
provides=('clvk' 'opencl-driver')
conflicts=('clvk')
options=('!lto' '!debug')
install="$pkgname.install"
source=("git+$url.git"
        'git+https://github.com/google/clspv.git'
        'opencl-headers::git+https://github.com/KhronosGroup/OpenCL-Headers.git'
        'spirv-headers::git+https://github.com/KhronosGroup/SPIRV-Headers.git'
        'spirv-llvm-translator::git+https://github.com/KhronosGroup/SPIRV-LLVM-Translator.git'
        'spirv-tools::git+https://github.com/KhronosGroup/SPIRV-Tools.git'
        'clspv.patch::https://github.com/google/clspv/commit/7e2a07a8c337fd7beb3b53873094bdaceb928b1d.diff'
        'clvk-install-paths.patch')
sha256sums=('SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            '7c7bc20f52abdb144fcbaa52d2791f2137f42f053e292c362cea2c3c4bd3e2db'
            '34ae444868e7d1c0c7941c73d37f15c01058269c30a8296b056d391e7f8168f2')

pkgver() {
    cd "$srcdir/${pkgname::-4}"
    printf 'r%s.%s' "$(git rev-list --count HEAD)" "$(git rev-parse --short=7 HEAD)"
}

prepare() {
    cd "$srcdir/${pkgname::-4}"
    git submodule init
    for module in clspv opencl-headers spirv-headers spirv-llvm-translator spirv-tools; do
        git config "submodule.$module.url" "$srcdir/$module"
    done
    git -c protocol.file.allow=always submodule update
    patch -d "$srcdir/${pkgname::-4}" -Np1 < "$srcdir/clvk-install-paths.patch"

    cd "$srcdir/${pkgname::-4}/external/clspv"
    if ! grep -q clspv_local lib/FixupBuiltinsPass.cpp; then
        patch -Np1 < "$srcdir/clspv.patch"
    fi

    cd "$srcdir/${pkgname::-4}/external/clspv/utils"
    python fetch_sources.py --shallow
}

build() {
    local source_prefix="/usr/src/debug/$pkgname"

    cmake -B "$srcdir/build" -S "$srcdir/${pkgname::-4}" \
        -GNinja \
        -DCMAKE_BUILD_TYPE=Release \
        -DCLVK_DEFAULT_CLSPV_BINARY_PATH='/usr/lib/clvk/clspv' \
        -DCLVK_DEFAULT_LLVMSPIRV_BINARY_PATH='/usr/bin/llvm-spirv' \
        -DLLVM_FORCE_VC_REPOSITORY='https://github.com/llvm/llvm-project' \
        -DCMAKE_C_FLAGS="$CFLAGS -ffile-prefix-map=$srcdir=$source_prefix" \
        -DCMAKE_CXX_FLAGS="$CXXFLAGS -ffile-prefix-map=$srcdir=$source_prefix" \
        -DCMAKE_EXE_LINKER_FLAGS="$LDFLAGS" \
        -DCLVK_BUILD_SPIRV_TOOLS=OFF \
        -DSKIP_SPIRV_TOOLS_INSTALL=ON \
        -DCLSPV_BUILD_TESTS=OFF \
        -DCLVK_BUILD_TESTS=OFF \
        -DCMAKE_INSTALL_PREFIX="/usr/lib/${pkgname::-4}"
    cmake --build "$srcdir/build"
}

package() {
    DESTDIR="$pkgdir" cmake --install "$srcdir/build"
    install -dm755 "$pkgdir/etc/OpenCL/vendors" "$pkgdir/etc/profile.d"

    printf '%s\n' '/usr/lib/clvk/libOpenCL.so' > "$pkgdir/etc/OpenCL/vendors/clvk64.icd"
    printf '%s\n' 'export CLVK_CONFIG_FILE=/usr/lib/clvk/clvk.conf' \
        > "$pkgdir/etc/profile.d/clvk.sh"
    printf '%s\n' 'clspv_path = /usr/lib/clvk/clspv' 'clspv_options = -w' \
        > "$pkgdir/usr/lib/clvk/clvk.conf"
}
