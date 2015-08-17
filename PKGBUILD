# Maintainer: -
# Contributors: Thomas, Det, Cian Mc Govern <cian@cianmcgovern.com>, Ng Oon-Ee <ngoonee.talk@gmail.com>
# Based on nvidia-beta-all: https://aur.archlinux.org/packages/nvidia-beta-all/

pkgname=nvidia-all
pkgver=346.47
pkgrel=1
pkgdesc="NVIDIA drivers for all kernels"
arch=('i686' 'x86_64')
url="http://www.nvidia.com/"
license=('custom:NVIDIA')
depends=('nvidia-libgl' "nvidia-utils=$pkgver")
makedepends=('linux-headers' 'pacman>=4.2.0')
provides=('nvidia')
conflicts=('nvidia-96xx' 'nvidia-173xx' 'nvidia')
options=('!strip')
install=$pkgname.install

# Installer name
case "$CARCH" in
  x86_64) _pkg="NVIDIA-Linux-x86_64-$pkgver-no-compat32" ;;
  i686)   _pkg="NVIDIA-Linux-x86-$pkgver" ;;
esac

# Source
source_x86_64=("http://us.download.nvidia.com/XFree86/Linux-x86_64/$pkgver/NVIDIA-Linux-x86_64-$pkgver-no-compat32.run")
source_i686=("http://us.download.nvidia.com/XFree86/Linux-x86/$pkgver/NVIDIA-Linux-x86-$pkgver.run")
md5sums_x86_64=('115b5b2d136c4b44c658ef823b8a4bab')
md5sums_i686=('ae61b6c3c081383f991bcc64ee0844b1')

# Patch
source=('nvidia-linux-3.18.patch')
md5sums=('ff8a5f979e4428f8c847423fb007042c')

prepare() {
  # Remove previous builds
  if [[ -d $_pkg ]]; then
    rm -rf $_pkg
  fi

  # Extract
  msg2 "Self-Extracting $_pkg.run..."
  sh $_pkg.run -x
  cd $_pkg

  for _kernel in $(cat /usr/lib/modules/extramodules-*/version); do
    # Use separate source directories
    cp -r kernel kernel-$_kernel

    # Patch
    if [[ $(ls "$srcdir"/*.patch 2>/dev/null) ]]; then
      cd kernel-$_kernel

      # Loop
      for _patch in "$srcdir"/*.patch; do
        _major_patch=$(echo $_patch | grep -Po "\d+\.\d+")

        # Check version
        if (( $(vercmp $_kernel $_major_patch) >= 0 )); then
          msg2 "Applying ${_patch##*/} for $_kernel..."
          patch -p2 -i "$_patch"
        fi
      done

      cd ..
    fi
  done
}


build() {
  # Build for all kernels
  for _kernel in $(cat /usr/lib/modules/extramodules-*/version); do
    cd "$srcdir"/$_pkg/kernel-$_kernel

    # Main module
    msg2 "Building Nvidia module for $_kernel..."
    make SYSSRC=/usr/lib/modules/$_kernel/build module

    # Unified memory: http://devblogs.nvidia.com/parallelforall/unified-memory-in-cuda-6/
    if [[ $CARCH = x86_64 ]]; then
      cd uvm
      msg2 "Building Unified memory module for $_kernel..."
      make SYSSRC=/usr/lib/modules/$_kernel/build module
    fi
  done
}

package() {
  # Install for all kernels
  for _extramod in $(find /usr/lib/modules/extramodules-*/version -printf '%h\n'); do
    _kernel=$(cat $_extramod/version)

    # Install
    install -Dm644 $_pkg/kernel-$_kernel/nvidia.ko \
           "$pkgdir"/$_extramod/nvidia.ko

    # Unified Memory
    if [[ $CARCH = x86_64 ]]; then
      install -Dm644 $_pkg/kernel-$_kernel/uvm/nvidia-uvm.ko \
            "$pkgdir/$_extramod/nvidia-uvm.ko"
    fi

    # Compress
    gzip "$pkgdir"/$_extramod/nvidia*.ko
  done

  # Blacklist Nouveau
  install -d "$pkgdir"/usr/lib/modprobe.d/
  echo "blacklist nouveau" >> "$pkgdir"/usr/lib/modprobe.d/nvidia.conf
}
