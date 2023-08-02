# Maintainer: Laurent Carlier <lordheavym@gmail.com>
# Maintainer: Solomon Choina <shlomochoina@gmail.com>
# Contributor: Jan de Groot <jgc@archlinux.org>
# Contributor: Andreas Radke <andyrtr@archlinux.org>

pkgbase=mesa-git
pkgname=('vulkan-mesa-layers-git' 'opencl-mesa-git' 'vulkan-intel-git' 'vulkan-radeon-git' 'vulkan-swrast-git' 'vulkan-panfrost-git' 'vulkan-virtio-git' 'mesa-git')
pkgdesc="mesa trunk (git version)"
epoch=1
pkgver=23.3.0_devel.175025.61d24ea88ce
pkgrel=1
groups=('chaotic-mesa-git')
arch=('x86_64')

makedepends=('python-mako' 'libxml2' 'libx11' 'libdrm' 'xorgproto' 'libxrandr' 'valgrind'
	     'libxshmfence' 'libxxf86vm' 'libxdamage' 'libvdpau' 'libva' 'libxv'
             'polly-git' 'wayland' 'wayland-protocols' 'elfutils' 'llvm-git' 'llvm-libs-git' 'systemd' 'libsystemd'
	     'libomxil-bellagio' 'libglvnd' 'libunwind' 'lm_sensors' 'libxvmc'
             'libepoxy' 'gtk3' 'python-ply'
             'meson' 'libclc-git' 'clang-git' 'glslang' 'zstd' 'vulkan-icd-loader' 'git' 'directx-headers' 'spirv-llvm-translator-git')
makedepends+=('rust-nightly' 'spirv-tools' 'rust-bindgen')
url="https://mesa3d.org"
license=('custom')
source=('mesa::git+https://gitlab.freedesktop.org/mesa/mesa.git'
        'LICENSE')

sha256sums=('SKIP'
            '7052ba73bb07ea78873a2431ee4e828f4e72bda7d176d07f770fa48373dec537')

pkgver() {
  cd "$srcdir"/mesa

  read -r _ver <VERSION
  echo "${_ver/-/_}.$(git rev-list --count HEAD).$(git rev-parse --short HEAD)"
}

_libdir=usr/lib

build() {
  local meson_options=(
    --libdir=/"$_libdir"
    -D b_lto=false
    -D cpp_std=c++17
    -D rust_std=2021
    -D b_ndebug=true
    -D platforms=x11,wayland
    -D gallium-drivers=r300,r600,radeonsi,nouveau,iris,zink,virgl,svga,swrast,panfrost,lima,asahi,crocus,d3d12
    -D vulkan-drivers=amd,intel,intel_hasvk,swrast,virtio,panfrost
    -D vulkan-layers=device-select,intel-nullhw,overlay
    -D dri3=enabled
    -D egl=enabled
    -D gallium-extra-hud=true
    -D gallium-nine=true
    -D gallium-omx=bellagio
    -D gallium-opencl=disabled
    -D gallium-rusticl=true
    -D gallium-va=enabled
    -D gallium-vdpau=enabled
    -D gallium-xa=enabled
    -D android-libbacktrace=disabled
    -D gbm=enabled
    -D gles1=disabled
    -D gles2=enabled
    -D opencl-spirv=true
    -D glvnd=true
    -D glx=dri
    -D libunwind=enabled
    -D llvm=enabled
    -D lmsensors=enabled
    -D osmesa=true
    -D shared-glapi=enabled
    -D microsoft-clc=disabled
    -D valgrind=enabled
    -D zstd=enabled
    -D xlib-lease=enabled
    -D intel-xe-kmd=enabled
    -D video-codecs=h264dec,h264enc,h265enc,h265dec
  )

  arch-meson mesa build "${meson_options[@]}"
  # Print config
  meson configure build

  ninja -C build

  # fake installation to be seperated into packages
  # outside of fakeroot but mesa doesn't need to chown/mod
  DESTDIR="$srcdir/fakeinstall" ninja -C build install
}

_install() {
  local src f dir
  for src; do
    f="${src#fakeinstall/}"
    dir="$pkgdir/${f%/*}"
    install -m755 -d "$dir"
    mv -v "$src" "$dir/"
  done
}

package_vulkan-mesa-layers-git() {
  pkgdesc="Mesa's Vulkan overlay layers (git version)"
  provides=('vulkan-mesa-layers')
  depends=('wayland' 'libxcb' 'python' 'libdrm')
  conflicts=('vulkan-mesa-layer-git' 'vulkan-mesa-layers')
  replaces=('vulkan-mesa-layer-git')

  _install fakeinstall/usr/share/vulkan/explicit_layer.d
  _install fakeinstall/usr/share/vulkan/implicit_layer.d

  _install fakeinstall/"$_libdir"/libVkLayer_*.so
  _install fakeinstall/usr/bin/mesa-overlay-control.py

  install -m644 -Dt "$pkgdir/usr/share/licenses/$pkgname" LICENSE
}

package_opencl-mesa-git() {
  pkgdesc="OpenCL support for mesa drivers (git version)"
  depends=('libdrm' 'libclc-git' 'clang-git' 'expat')
  optdepends=('opencl-headers: headers necessary for OpenCL development')
  provides=('opencl-mesa' 'opencl-driver')
  conflicts=('opencl-mesa' 'opencl-clover-mesa' 'opencl-rusticl-mesa')

  _install fakeinstall/etc/OpenCL
  _install fakeinstall/"$_libdir"/lib*OpenCL*

  install -m644 -Dt "$pkgdir/usr/share/licenses/$pkgname" LICENSE
}

package_vulkan-intel-git() {
  pkgdesc="Intel's Vulkan mesa driver (git version)"
  depends=('wayland' 'libx11' 'libxshmfence' 'libdrm' 'zstd' 'systemd-libs' 'llvm-libs-git')
  optdepends=('vulkan-mesa-layer-git: additional vulkan layers')
  provides=('vulkan-intel' 'vulkan-driver')
  conflicts=('vulkan-intel')

  _install fakeinstall/usr/share/vulkan/icd.d/intel_*.json
  _install fakeinstall/"$_libdir"/libvulkan_intel*.so

  install -m644 -Dt "$pkgdir/usr/share/licenses/$pkgname" LICENSE
}

package_vulkan-radeon-git() {
  pkgdesc="Radeon's Vulkan mesa driver (git version)"
  depends=('wayland' 'libx11' 'libxshmfence' 'libelf' 'libdrm' 'llvm-libs-git' 'systemd-libs')
  optdepends=('vulkan-mesa-layer-git: additional vulkan layers')
  provides=('vulkan-radeon' 'vulkan-driver')
  conflicts=('vulkan-radeon')

  _install fakeinstall/usr/share/vulkan/icd.d/radeon_icd*.json
  _install fakeinstall/"$_libdir"/libvulkan_radeon.so

  install -m644 -Dt "$pkgdir/usr/share/licenses/$pkgname" LICENSE
}

package_vulkan-swrast-git() {
  pkgdesc="Vulkan software rasteriser driver (git version)"
  depends=('wayland' 'libx11' 'libxshmfence' 'libdrm' 'zstd' 'llvm-libs-git' 'libunwind')
  optdepends=('vulkan-mesa-layers-git: additional vulkan layers')
  provides=('vulkan-swrast' 'vulkan-driver')
  conflicts=('vulkan-swrast' 'vulkan-mesa-git')
  replaces=('vulkan-mesa-git')

  _install fakeinstall/usr/share/vulkan/icd.d/lvp_icd*.json
  _install fakeinstall/"$_libdir"/libvulkan_lvp.so

  install -m644 -Dt "$pkgdir/usr/share/licenses/$pkgname" LICENSE
}

package_vulkan-panfrost-git() {
  pkgdesc="Arm Mali's vulkan mesa driver (git version)"
  depends=('wayland' 'libx11' 'libxshmfence' 'libdrm' 'zstd' 'llvm-libs-git' 'libunwind')
  optdepends=('vulkan-mesa-layers-git: additional vulkan layers')
  provides=('vulkan-panfrost' 'vulkan-driver')
  conflicts=('vulkan-panfrost')

  _install fakeinstall/usr/share/vulkan/icd.d/panfrost_*.json
  _install fakeinstall/usr/lib/libvulkan_panfrost*.so

  install -m644 -Dt "$pkgdir/usr/share/licenses/$pkgname" LICENSE
}

package_vulkan-virtio-git() {
  pkgdesc="Venus Vulkan mesa driver for Virtual Machines (git version)"
depends=(
    'libdrm'
    'libx11'
    'libxshmfence'
    'systemd'
    'wayland'
    'xcb-util-keysyms'
    'zstd'
  )
  optdepends=('vulkan-mesa-layers-git: additional vulkan layers')
  provides=('vulkan-driver')

  _install fakeinstall/usr/share/vulkan/icd.d/virtio_icd*.json
  _install fakeinstall/$_libdir/libvulkan_virtio.so

  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" LICENSE
}

package_mesa-git() {
  pkgdesc="an open-source implementation of the OpenGL specification (git version)"
  depends=('libdrm' 'wayland' 'libxxf86vm' 'libxdamage' 'libxshmfence' 'libsystemd' 'libelf'
           'libomxil-bellagio' 'libunwind' 'llvm-libs-git' 'lm_sensors' 'libglvnd' 'libxv' 'libxvmc')
  optdepends=('opengl-man-pages: for the OpenGL API man pages')
  provides=('mesa' 'mesa-vdpau' 'libva-mesa-driver' 'mesa-libgl' 'opengl-driver')
  conflicts=('mesa' 'mesa-vdpau' 'libva-mesa-driver' 'mesa-libgl')

  # libva-mesa-driver
  _install fakeinstall/"$_libdir"/dri/*_drv_video.so
  # mesa-vdpau
  _install fakeinstall/usr/lib/vdpau
  _install fakeinstall/usr/share/drirc.d/00-mesa-defaults.conf
  _install fakeinstall/usr/share/drirc.d/00-radv-defaults.conf
  _install fakeinstall/usr/share/glvnd/egl_vendor.d/50_mesa.json

  # ati-dri, nouveau-dri, intel-dri, svga-dri, swrast
  _install fakeinstall/"$_libdir"/dri/*_dri.so

  _install fakeinstall/usr/lib/bellagio
  _install fakeinstall/usr/lib/d3d
  _install fakeinstall/usr/lib/lib{gbm,glapi}.so*
  _install fakeinstall/usr/lib/libOSMesa.so*
  _install fakeinstall/usr/lib/libxatracker.so*
  # _install fakeinstall/usr/lib/libswrAVX*.so*

  # in vulkan-headers
  rm -rfv fakeinstall/usr/include/vulkan

  _install fakeinstall/usr/include
  _install fakeinstall/usr/lib/pkgconfig

  # libglvnd support
  _install fakeinstall/usr/lib/libGLX_mesa.so*
  _install fakeinstall/usr/lib/libEGL_mesa.so*

  # indirect rendering
  ln -s /usr/lib/libGLX_mesa.so.0 "$pkgdir/usr/lib/libGLX_indirect.so.0"

  # make sure there are no files left to install
  find fakeinstall -depth -print0 | xargs -0 rmdir

  install -m644 -Dt "$pkgdir/usr/share/licenses/$pkgname" LICENSE
}
