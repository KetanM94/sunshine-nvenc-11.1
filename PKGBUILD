pkgname=sunshine-git
pkgver=2024.823.131748.r0.gc9f853d
pkgrel=1
pkgdesc="Game Stream server for Moonlight, latest git (NvENC API 11.1 for driver 470.xx)"
arch=('x86_64')
url=https://github.com/LizardByte/Sunshine
license=('GPL3')
install=sunshine-git.install
options=('!debug')

depends=(
  avahi
  curl
  libayatana-appindicator
  libevdev
  libmfx
  libnotify
  libpulse
  libva
  libvdpau
  libx11
  libxcb
  libxfixes
  libxrandr
  libxtst
  miniupnpc
  numactl
  openssl
  opus
  udev
)

makedepends=(
  git
  cmake
  ninja
  nodejs
  npm
)

optdepends=(
  'cuda-11.1: NvFBC capture support'
  'libcap'
  'libdrm'
)

provides=(sunshine)
conflicts=(sunshine)

source=(
  git+https://github.com/LizardByte/Sunshine.git#branch=master
  moonlight-common-c::git+https://github.com/moonlight-stream/moonlight-common-c.git
  Simple-Web-Server::git+https://gitlab.com/eidheim/Simple-Web-Server.git
  ViGEmClient::git+https://github.com/LizardByte/Virtual-Gamepad-Emulation-Client.git
  nv-codec-headers::git+https://github.com/FFmpeg/nv-codec-headers.git#branch=sdk/11.1
  TPCircularBuffer::git+https://github.com/michaeltyson/TPCircularBuffer.git
  build-deps::git+https://github.com/KetanM94/sunshine-build-deps.git#branch=dist
  nanors::git+https://github.com/sleepybishop/nanors.git
  enet::git+https://github.com/cgutman/enet.git
  inputtino::git+https://github.com/games-on-whales/inputtino.git
  nvapi-open-source-sdk::git+https://github.com/LizardByte/nvapi-open-source-sdk.git
  tray::git+https://github.com/LizardByte/tray.git
  wayland-protocols::git+https://gitlab.freedesktop.org/wayland/wayland-protocols.git
  wlr-protocols::git+https://gitlab.freedesktop.org/wlroots/wlr-protocols.git
  00-nvenc_base-api-11.1-support.patch
  01-add-nvencode-api-vui-params.patch
)

sha256sums=('SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP')

pkgver() {
  cd Sunshine
  git describe --long --tags --abbrev=7 | sed 's/^v//;s/\([^-]*-g\)/r\1/;s/-/./g'
}

prepare() {
  cd Sunshine
  echo "Initializing and updating submodules..."
  for module in moonlight-common-c Simple-Web-Server ViGEmClient nv-codec-headers TPCircularBuffer build-deps nanors inputtino nvapi-open-source-sdk tray wayland-protocols wlr-protocols; do
    echo "Initializing and updating submodule $module..."
    for src in "${source[@]}"; do
      if [[ "$src" =~ ^$module:: ]]; then
        url=${src#*git+}
        url=${url%%\#*}
        echo "Updating submodule URL for $module to $url"
        git config -f .gitmodules submodule.third-party/$module.url "$url"
        if [[ "$src" =~ \# ]]; then
          branch=${src##*\#branch=}
          echo "Updating submodule branch for $module to $branch"
          git config -f .gitmodules submodule.third-party/$module.branch "${branch%%=*}"
        fi
      fi
    done
    git submodule init third-party/$module
    git submodule update --remote --reference "$srcdir/$module" third-party/$module
  done

  pushd third-party/moonlight-common-c
  echo "Initializing and updating enet submodule..."
  git submodule init enet
  git submodule update --reference ${srcdir}/enet enet
  popd

  echo "Applying patches..."
  for patch in "${source[@]}"; do
    if [[ "$patch" == *.patch ]]; then
      echo "Applying patch $patch"
      patch -Np1 -i "$srcdir"/"$patch" || true
    fi
  done
}

build() {
  pushd Sunshine
  npm install
  popd
  
  export BRANCH="master"
  export BUILD_VERSION="${pkgver}"
  export COMMIT="$(git -C "$_pkgsrc" rev-parse HEAD)"
  
  export CFLAGS="${CFLAGS/-Werror=format-security/}"
  export CXXFLAGS="${CXXFLAGS/-Werror=format-security/}"
  export LDFLAGS="${LDFLAGS} -L/usr/lib -lstdc++"

  echo "Building Sunshine..."
  cmake -B build_dir -S Sunshine -W no-dev -G Ninja \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_CUDA_COMPILER="/opt/cuda/bin/nvcc" \
    -DCMAKE_CUDA_HOST_COMPILER="/usr/bin/gcc-10" \
    -DCMAKE_INSTALL_PREFIX=/usr \
    -DSUNSHINE_EXECUTABLE_PATH=/usr/bin/sunshine \
    -DSUNSHINE_ASSETS_DIR="share/sunshine" \
    -DBUILD_TESTS=0 \
    -DBUILD_DOCS=0

  cmake --build build_dir
}

package() {
  echo "Installing Sunshine..."
  DESTDIR="${pkgdir}" cmake --install build_dir
}
