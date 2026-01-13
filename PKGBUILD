# Maintainer: Lubosz Sarnecki <lubosz@gmail.com>
# Contributor: acxz <akashpatel2008 at yahoo dot com>
# Contributor: Sven-Hendrik Haase <svenstaro@archlinux.org>
# Contributor: Konstantin Gizdov (kgizdov) <arch@kge.pw>
# Contributor: Adria Arrufat (archdria) <adria.arrufat+AUR@protonmail.ch>
# Contributor: Thibault Lorrain (fredszaq) <fredszaq@gmail.com>

pkgbase=tensorflow-rocm

# Flags for building without/with cpu optimizations
_build_no_opt=1
_build_opt=1

pkgname=()
[ "$_build_no_opt" -eq 1 ] && pkgname+=(tensorflow-rocm python-tensorflow-rocm)
[ "$_build_opt" -eq 1 ] && pkgname+=(tensorflow-opt-rocm python-tensorflow-opt-rocm)

pkgver=2.20.0
_pkgver=2.20.0
pkgrel=1
pkgdesc="Library for computation using data flow graphs for scalable machine learning"
url="https://www.tensorflow.org/"
license=('APACHE')
arch=('x86_64')
depends=('c-ares' 'pybind11' 'openssl' 'libpng' 'curl' 'giflib' 'icu' 'libjpeg-turbo' 'intel-oneapi-openmp'
         'intel-oneapi-compiler-shared-runtime-libs')
makedepends=('bazel' 'python-numpy' 'rocm-hip-sdk' 'roctracer' 'rccl' 'git' 'miopen' 'python-wheel' 'openmp'
             'python-installer' 'python-setuptools' 'python-h5py' 'python-keras-applications'
             'python-keras-preprocessing' 'cython' 'patchelf' 'python-requests' 'libxcrypt-compat' 'clang20')
optdepends=('tensorboard: Tensorflow visualization toolkit')
source=("$pkgname-$pkgver.tar.gz::https://github.com/tensorflow/tensorflow/archive/v${_pkgver}.tar.gz"
        https://github.com/bazelbuild/bazel/releases/download/7.4.1/bazel_nojdk-7.4.1-linux-x86_64
        tensorflow-2.19.0-tf-runtime-workspace.patch
        tf-runtime-forward-decls.patch
        tensorflow-2.19.0-matmul-it-unused-result.patch
        protobuf-deps.patch
        tensorflow-2.20.0-python-3.14.patch)
sha512sums=('e1e27c84491a28dc5b7deb181de0fbad27ddd58cdd7ee2f6815bebd26d7ff400a94efea52eb7da344702adcd9181a474a76dc9e94d2ad7d6511d261deffa0cf5'
            'ca01422266ed741a7d2ec1fdaba8fed8aa1d1ca4d53e45345225403cb132dc56266fd799351e80aeb50df6f2f4e29e6e0dc9430aef882d749425e8521ba0a806'
            '27a07c1f5ab3898e049717e3f56498577c62840416b425e582a308d22112c0341b26a69d419daa0f80fac5be53cee122122b1b8200fc885ae5f7346b8e3ecb10'
            'b3ac22ede074decdb751adbc8f0324f4b729ffb4927fddcc67250e2f2a2812d2ed6ff515ed98cb3eaf5c1e4b755d6b780f5515181b6f10f046dbaa91e0eb1964'
            'e5856d277edcef6ab655e8b641f1ec8fe9823264221d687c9938bb00d5acee626a2a05b46cfd97fbad8551f270332ed09b73848bf9784137317fee47ad64c80e'
            '5a90a66ea6b78196aaa8852f90ac2af82cfac12b4f7eefa9b1e04ff0fbefd4fdc82f9705f77634efe76c55d5e1804cdfd736dd8ceb875d7ce2757552467a04dd'
            'cfb1eced1f4b4534deddf14b73d45ae4f8104264ccada1a8854e0dbf7ef2a2f28e67930348211541630a102994ad258d55804359448d225863c4bd5c8762fff1')

# consolidate common dependencies to prevent mishaps
_common_py_depends=(python-termcolor python-astor python-gast python-numpy python-protobuf absl-py
                    python-h5py python-keras python-opt_einsum python-wrapt
                    python-astunparse python-pasta python-flatbuffers python-typing_extensions python-ml-dtypes)

get_pyver () {
  python -c 'import sys; print(str(sys.version_info[0]) + "." + str(sys.version_info[1]))'
}

check_dir() {
  # first make sure we do not break parsepkgbuild
  if ! command -v cp &> /dev/null; then
    >&2 echo "'cp' command not found. PKGBUILD is probably being checked by parsepkgbuild."
    if ! command -v install &> /dev/null; then
      >&2 echo "'install' command also not found. PKGBUILD must be getting checked by parsepkgbuild."
      >&2 echo "Cannot check if directory '${1}' exists. Ignoring."
      >&2 echo "If you are not running nacmap or parsepkgbuild, please make sure the PATH is correct and try again."
      >&2 echo "PATH should not be '/dummy': PATH=$PATH"
      return 0
    fi
  fi
  # if we are running normally, check the given path
  if [ -d "${1}" ]; then
    return 0
  else
    >&2 echo Directory "${1}" does not exist or is a file! Exiting...
    exit 1
  fi
}

prepare() {
  # Since Tensorflow is currently incompatible with our version of Bazel, we're going to use
  # their exact version of Bazel to fix that. Stupid problems call for stupid solutions.
  install -Dm755 "${srcdir}"/bazel_nojdk-7.4.1-linux-x86_64 bazel/bazel
  export PATH="${srcdir}/bazel:$PATH"
  bazel --version

  # Custom patch for python 3.14 support
  patch -Np1 -i ../tensorflow-2.20.0-python-3.14.patch -d tensorflow-${_pkgver}

  # Custom patch for the @tf_runtime external dependency
  patch -Np1 -i ../tensorflow-2.19.0-tf-runtime-workspace.patch -d tensorflow-${_pkgver}
  cp tf-runtime-forward-decls.patch tensorflow-${_pkgver}/third_party/tf_runtime/forward_decls.patch

  # Custom patch for error: ignoring return value of function declared with 'nodiscard' attribute [-Werror,-Wunused-result]
  patch -Np1 -i ../tensorflow-2.19.0-matmul-it-unused-result.patch -d tensorflow-${_pkgver}

  # Get rid of hardcoded versions. Not like we ever cared about what upstream
  # thinks about which versions should be used anyway. ;) (FS#68772)
  sed -i -E "s/'([0-9a-z_-]+) .= [0-9].+[0-9]'/'\1'/" tensorflow-${_pkgver}/tensorflow/tools/pip_package/setup.py.tpl

  # building the C/C++ interface requires more work but we cannot build tensorflow and python-tensorflow together
  # https://gitlab.archlinux.org/archlinux/packaging/packages/tensorflow/-/issues/25

  # Disable PYWRAP_RULES to let the libtensorflow.so etc. targets be found
  sed -i '/build --repo_env=USE_PYWRAP_RULES=True/d' tensorflow-${_pkgver}/.bazelrc tensorflow-${_pkgver}/third_party/xla/tensorflow.bazelrc

  # Patch protobuf dependencies in bazel
  # https://gitlab.archlinux.org/archlinux/packaging/packages/tensorflow/-/issues/24#note_334439
  patch -Np1 -i ../protobuf-deps.patch -d tensorflow-${_pkgver}

  cp -r tensorflow-${_pkgver} tensorflow-${_pkgver}-rocm
  cp -r tensorflow-${_pkgver} tensorflow-${_pkgver}-opt-rocm

  # These environment variables influence the behavior of the configure call below.
  export TF_NEED_JEMALLOC=1
  export TF_NEED_KAFKA=1
  export TF_NEED_OPENCL_SYCL=0
  export TF_NEED_AWS=1
  export TF_NEED_GCP=1
  export TF_NEED_HDFS=1
  export TF_NEED_S3=1
  export TF_ENABLE_XLA=1
  export TF_NEED_GDR=0
  export TF_NEED_VERBS=0
  export TF_NEED_OPENCL=0
  export TF_NEED_MPI=0
  export TF_NEED_TENSORRT=0
  export TF_NEED_NGRAPH=0
  export TF_NEED_IGNITE=0
  export TF_NEED_ROCM=1
  # Uncomment this when you want to specify specific ROCM_ARCH(s)
  # Otherwise tensorflow will automatically detect your architecture
  # See: https://github.com/tensorflow/tensorflow/commit/c04822a49d669f2d74a566063852243d993e18b1
  # export TF_ROCM_AMDGPU_TARGETS=gfx803,gfx900,gfx906,gfx908,gfx90a,gfx1030,gfx1100,gfx1101,gfx1102
  export TF_NEED_CLANG=1
  export CLANG_COMPILER_PATH=/usr/lib/llvm20/bin/clang
  # See https://github.com/tensorflow/tensorflow/blob/master/third_party/systemlibs/syslibs_configure.bzl
  export TF_SYSTEM_LIBS="boringssl,curl,cython,gif,icu,libjpeg_turbo,nasm,png,zlib"
  export TF_SET_ANDROID_WORKSPACE=0
  export TF_DOWNLOAD_CLANG=0
  # export TF_NCCL_VERSION=$(pkg-config nccl --modversion | grep -Po '\d+\.\d+')
  export NCCL_INSTALL_PATH=/usr
  # Does tensorflow really need the compiler overridden in 5 places? Yes.
  # https://github.com/tensorflow/tensorflow/issues/60577
  export CC=gcc
  export CXX=g++
  export GCC_HOST_COMPILER_PATH=/usr/bin/gcc
  export HOST_C_COMPILER=/usr/bin/${CC}
  export HOST_CXX_COMPILER=/usr/bin/${CXX}
  export TF_CUDA_CLANG=0  # Clang currently disabled because it's not compatible at the moment.
  export CLANG_CUDA_COMPILER_PATH=/usr/lib/llvm20/bin/clang
  export TF_CUDA_PATHS=/opt/cuda,/usr/lib,/usr
  export TF_CUDA_VERSION=$(/opt/cuda/bin/nvcc --version | sed -n 's/^.*release \(.*\),.*/\1/p')
  export TF_CUDNN_VERSION=$(sed -n 's/^#define CUDNN_MAJOR\s*\(.*\).*/\1/p' /usr/include/cudnn_version.h)
  # https://github.com/tensorflow/tensorflow/blob/1ba2eb7b313c0c5001ee1683a3ec4fbae01105fd/third_party/gpus/cuda_configure.bzl#L411-L446
  # according to the above, we should be specifying CUDA compute capabilities as 'sm_XX' or 'compute_XX' from now on
  # add latest PTX for future compatibility
  # Valid values can be discovered from nvcc --help
  export TF_CUDA_COMPUTE_CAPABILITIES=sm_52,sm_53,sm_60,sm_61,sm_62,sm_70,sm_72,sm_75,sm_80,sm_86,sm_87,sm_89,sm_90,compute_90
  export TF_PYTHON_VERSION=$(get_pyver)
  export PYTHON_BIN_PATH=/usr/bin/python$(get_pyver)
  export USE_DEFAULT_PYTHON_LIB_PATH=1

  export BAZEL_ARGS="-c opt --verbose_failures --config=verbose_logs --copt=-Wno-gnu-offsetof-extensions"
}

build() {
  if [ "$_build_no_opt" -eq 1 ]; then
    echo "Building with rocm and without non-x86-64 optimizations"
    cd "${srcdir}"/tensorflow-${_pkgver}-rocm
    export CC_OPT_FLAGS="-march=x86-64"
    export TF_NEED_CUDA=0
    export TF_NEED_ROCM=1
    ./configure
    bazel \
      build --config=rocm \
        ${BAZEL_ARGS[@]} \
        //tensorflow:libtensorflow.so \
        //tensorflow:libtensorflow_cc.so \
        //tensorflow:libtensorflow_framework.so \
        //tensorflow:install_headers \
        //tensorflow/tools/pip_package:wheel --repo_env=WHEEL_NAME=tensorflow
  fi


  if [ "$_build_opt" -eq 1 ]; then
    echo "Building with rocm and with non-x86-64 optimizations"
    cd "${srcdir}"/tensorflow-${_pkgver}-opt-rocm
    export CC_OPT_FLAGS="-march=haswell -O2"
    export TF_NEED_CUDA=0
    export TF_NEED_ROCM=1
    ./configure
    bazel \
      build --config=opt --config=avx_linux --config=rocm \
        ${BAZEL_ARGS[@]} \
        //tensorflow:libtensorflow.so \
        //tensorflow:libtensorflow_cc.so \
        //tensorflow:libtensorflow_framework.so \
        //tensorflow:install_headers \
        //tensorflow/tools/pip_package:wheel --repo-env=WHEEL_NAME=tensorflow
  fi
}

_package() {
  # install headers first
  install -d "${pkgdir}"/usr/include/tensorflow
  cp -r bazel-bin/tensorflow/include/* "${pkgdir}"/usr/include/tensorflow/

  # install python-version to get all extra headers
  WHEEL_PACKAGE=$(find -L bazel-out -name "tensor*.whl")
  python -m installer --destdir="$pkgdir" $WHEEL_PACKAGE

  # move extra headers to correct location
  local _srch_path="${pkgdir}/usr/lib/python$(get_pyver)"/site-packages/tensorflow/include
  check_dir "${_srch_path}"  # we need to quit on broken search paths
  cp -rf "${_srch_path}"/* "${pkgdir}"/usr/include/tensorflow/

  # clean up unneeded files
  rm -rf "${pkgdir}"/usr/bin
  rm -rf "${pkgdir}"/usr/lib
  rm -rf "${pkgdir}"/usr/share

  # make sure no lib objects are outside valid paths
  local _so_srch_path="${pkgdir}/usr/include"
  check_dir "${_so_srch_path}"  # we need to quit on broken search paths
  find "${_so_srch_path}" -type f,l \( -iname "*.so" -or -iname "*.so.*" \) -print0 | while read -rd $'\0' _so_file; do
    # check if file is a dynamic executable
    ldd "${_so_file}" &>/dev/null && rm -rf "${_so_file}"
  done

  # pkgconfig
  tensorflow/c/generate-pc.sh --prefix=/usr --version=${pkgver}
  sed -e 's@/include$@/include/tensorflow@' -i tensorflow.pc -i tensorflow_cc.pc
  install -Dm644 tensorflow.pc "${pkgdir}"/usr/lib/pkgconfig/tensorflow.pc
  install -Dm644 tensorflow_cc.pc "${pkgdir}"/usr/lib/pkgconfig/tensorflow_cc.pc

  # .so files
  rm bazel-bin/tensorflow/*.params
  cp -P bazel-bin/tensorflow/*.so* "${pkgdir}"/usr/lib

  # C API headers
  install -Dm644 tensorflow/c/c_api.h "${pkgdir}"/usr/include/tensorflow/tensorflow/c/c_api.h

  # license
  install -Dm644 LICENSE "${pkgdir}"/usr/share/licenses/${pkgname}/LICENSE
}

_python_package() {
  WHEEL_PACKAGE=$(find -L bazel-ouot -name "tensor*.whl")
  python -m installer --destdir="$pkgdir" $WHEEL_PACKAGE

  # create symlinks to headers
  local _srch_path="${pkgdir}/usr/lib/python$(get_pyver)"/site-packages/tensorflow/include/
  check_dir "${_srch_path}"  # we need to quit on broken search paths
  find "${_srch_path}" -maxdepth 1 -mindepth 1 -type d -print0 | while read -rd $'\0' _folder; do
    rm -rf "${_folder}"
    _smlink="$(basename "${_folder}")"
    ln -s /usr/include/tensorflow/"${_smlink}" "${_srch_path}"
  done

  # tensorboard has been separated from upstream but they still install it with
  # tensorflow. I don't know what kind of sense that makes but we have to clean
  # it out from this pacakge.
  rm -r "${pkgdir}"/usr/bin/tensorboard

  install -Dm644 LICENSE "${pkgdir}"/usr/share/licenses/${pkgname}/LICENSE
}

package_tensorflow-rocm() {
  pkgdesc="Library for computation using data flow graphs for scalable machine learning (with ROCM)"
  depends+=(rocm-hip-sdk miopen rccl)
  conflicts=(tensorflow)
  provides=(tensorflow)

  cd "${srcdir}"/tensorflow-${_pkgver}-rocm
  _package
}

package_tensorflow-opt-rocm() {
  pkgdesc="Library for computation using data flow graphs for scalable machine learning (with ROCM and AVX CPU optimizations)"
  depends+=(rocm-hip-sdk miopen rccl)
  conflicts=(tensorflow)
  provides=(tensorflow tensorflow-rocm)

  cd "${srcdir}"/tensorflow-${_pkgver}-opt-rocm
  _package
}

package_python-tensorflow-rocm() {
  pkgdesc="Library for computation using data flow graphs for scalable machine learning (with ROCM)"
  depends+=(tensorflow-rocm rocm-hip-sdk miopen rccl "${_common_py_depends[@]}")
  conflicts=(python-tensorflow)
  provides=(python-tensorflow)

  cd "${srcdir}"/tensorflow-${_pkgver}-rocm
  _python_package
}

package_python-tensorflow-opt-rocm() {
  pkgdesc="Library for computation using data flow graphs for scalable machine learning (with ROCM and AVX CPU optimizations)"
  depends+=(tensorflow-opt-rocm rocm-hip-sdk miopen rccl "${_common_py_depends[@]}")
  conflicts=(python-tensorflow)
  provides=(python-tensorflow python-tensorflow-rocm)

  cd "${srcdir}"/tensorflow-${_pkgver}-opt-rocm
  _python_package
}

# vim:set ts=2 sw=2 et:

