# Maintainer: acxz <akashpatel2008 at yahoo dot com>
# Contributor: Sven-Hendrik Haase <svenstaro@gmail.com>
# Contributor: Konstantin Gizdov (kgizdov) <arch@kge.pw>
# Contributor: Adria Arrufat (archdria) <adria.arrufat+AUR@protonmail.ch>
# Contributor: Thibault Lorrain (fredszaq) <fredszaq@gmail.com>

pkgbase=tensorflow-rocm

# Flags for building without/with cpu optimizations
_build_no_opt=0
_build_opt=1

pkgname=()
[ "$_build_no_opt" -eq 1 ] && pkgname+=(tensorflow-rocm python-tensorflow-rocm)
[ "$_build_opt" -eq 1 ] && pkgname+=(tensorflow-opt-rocm python-tensorflow-opt-rocm)

pkgver=2.6.0
_pkgver=2.6.0
pkgrel=2
pkgdesc="Library for computation using data flow graphs for scalable machine learning"
url="https://www.tensorflow.org/"
license=('APACHE')
arch=('x86_64')
depends=('c-ares' 'intel-mkl' 'onednn' 'pybind11' 'openssl-1.0' 'lmdb' 'libpng' 'curl' 'giflib' 'icu' 'libjpeg-turbo')
makedepends=('bazel' 'python-numpy' 'rocm' 'rocm-libs' 'miopen' 'rccl' 'git'
             'python-pip' 'python-wheel' 'python-setuptools' 'python-h5py'
             'python-keras-applications' 'python-keras-preprocessing'
             'cython')
optdepends=('tensorboard: Tensorflow visualization toolkit')
source=("$pkgname-$pkgver.tar.gz::https://github.com/tensorflow/tensorflow/archive/v${_pkgver}.tar.gz"
        48935.patch
        fix-c++17-compat.patch
        build-against-actual-mkl.patch
        openssl-1.1.patch)

sha512sums=('d052da4b324f1b5ac9c904ac3cca270cefbf916be6e5968a6835ef3f8ea8c703a0b90be577ac5205edf248e8e6c7ee8817b6a1b383018bb77c381717c6205e05'
            '8a0fb7e728b144656503ee54b3c90483c619adf17b2081dceb2bd6bcd1435dd64afba97526d94114d4c10fc002d2d213ae6717ad407285b18e438b05fc1ed2ad'
            'f682368bb47b2b022a51aa77345dfa30f3b0d7911c56515d428b8326ee3751242f375f4e715a37bb723ef20a86916dad9871c3c81b1b58da85e1ca202bc4901e'
            'e51e3f3dced121db3a09fbdaefd33555536095584b72a5eb6f302fa6fa68ab56ea45e8a847ec90ff4ba076db312c06f91ff672e08e95263c658526582494ce08'
            'cb15e7331f62d6e77e1099055430cd026e5788f0cab202fbfad8e27c47fca9ad5e1467249683dcdaab8c76cab4dece016f8ecd0f0793adb256ff6d975f893125')


# consolidate common dependencies to prevent mishaps
_common_py_depends=(python-termcolor python-astor python-gast03 python-numpy python-protobuf absl-py python-h5py python-keras-applications python-keras-preprocessing python-tensorflow-estimator python-opt_einsum python-astunparse python-pasta python-flatbuffers)

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
  # Allow any bazel version
  echo "*" > tensorflow-${_pkgver}/.bazelversion

  # Tensorflow actually wants to build against a slimmed down version of Intel MKL called MKLML
  # See https://github.com/intel/mkl-dnn/issues/102
  # MKLML version that Tensorflow wants to use is https://github.com/intel/mkl-dnn/releases/tag/v0.21
  # patch -Np1 -d tensorflow-${_pkgver} -i "$srcdir"/build-against-actual-mkl.patch

  # https://github.com/tensorflow/tensorflow/pull/48935/files
  patch -p1 -d tensorflow-${_pkgver} -i "$srcdir"/48935.patch

  # https://bugs.archlinux.org/task/71597
  patch -p1 -d tensorflow-${_pkgver} -i "$srcdir"/openssl-1.1.patch

  # Get rid of hardcoded versions. Not like we ever cared about what upstream
  # thinks about which versions should be used anyway. ;) (FS#68772)
  sed -i -E "s/'([0-9a-z_-]+) .= [0-9].+[0-9]'/'\1'/" tensorflow-${_pkgver}/tensorflow/tools/pip_package/setup.py

  cp -r tensorflow-${_pkgver} tensorflow-${_pkgver}-rocm
  cp -r tensorflow-${_pkgver} tensorflow-${_pkgver}-opt-rocm

  # These environment variables influence the behavior of the configure call below.
  export PYTHON_BIN_PATH=/usr/bin/python
  export USE_DEFAULT_PYTHON_LIB_PATH=1
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
  export TF_ROCM_AMDGPU_TARGETS=gfx803
  # See https://github.com/tensorflow/tensorflow/blob/master/third_party/systemlibs/syslibs_configure.bzl
  export TF_SYSTEM_LIBS="boringssl,curl,cython,gif,icu,libjpeg_turbo,lmdb,nasm,png,pybind11,zlib"
  export TF_SET_ANDROID_WORKSPACE=0
  export TF_DOWNLOAD_CLANG=0
  export TF_NCCL_VERSION=$(pkg-config nccl --modversion | grep -Po '\d+\.\d+')
  export TF_IGNORE_MAX_BAZEL_VERSION=1
  export TF_MKL_ROOT=/opt/intel/mkl
  export NCCL_INSTALL_PATH=/usr
  export GCC_HOST_COMPILER_PATH=/usr/bin/gcc
  export HOST_C_COMPILER=/usr/bin/gcc
  export HOST_CXX_COMPILER=/usr/bin/g++
  export TF_CUDA_CLANG=0  # Clang currently disabled because it's not compatible at the moment.
  export CLANG_CUDA_COMPILER_PATH=/usr/bin/clang
  export TF_CUDA_PATHS=/opt/cuda,/usr/lib,/usr
  export TF_CUDA_VERSION=$(/opt/cuda/bin/nvcc --version | sed -n 's/^.*release \(.*\),.*/\1/p')
  export TF_CUDNN_VERSION=$(sed -n 's/^#define CUDNN_MAJOR\s*\(.*\).*/\1/p' /usr/include/cudnn_version.h)
  # https://github.com/tensorflow/tensorflow/blob/1ba2eb7b313c0c5001ee1683a3ec4fbae01105fd/third_party/gpus/cuda_configure.bzl#L411-L446
  # according to the above, we should be specifying CUDA compute capabilities as 'sm_XX' or 'compute_XX' from now on
  # add latest PTX for future compatibility
  export TF_CUDA_COMPUTE_CAPABILITIES=sm_52,sm_53,sm_60,sm_61,sm_62,sm_70,sm_72,sm_75,sm_80,sm_86,compute_86

  export CC=gcc
  export CXX=g++

  export BAZEL_ARGS="--config=mkl -c opt"
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
      build \
        ${BAZEL_ARGS[@]} \
        //tensorflow:libtensorflow.so \
        //tensorflow:libtensorflow_cc.so \
        //tensorflow:install_headers \
        //tensorflow/tools/pip_package:build_pip_package
    bazel-bin/tensorflow/tools/pip_package/build_pip_package --gpu "${srcdir}"/tmprocm
  fi


  if [ "$_build_opt" -eq 1 ]; then
    echo "Building with rocm and with non-x86-64 optimizations"
    cd "${srcdir}"/tensorflow-${_pkgver}-opt-rocm
    export CC_OPT_FLAGS="-march=haswell -O3"
    export TF_NEED_CUDA=0
    export TF_NEED_ROCM=1
    ./configure
    bazel \
      build --config=avx2_linux \
        ${BAZEL_ARGS[@]} \
        //tensorflow:libtensorflow.so \
        //tensorflow:libtensorflow_cc.so \
        //tensorflow:install_headers \
        //tensorflow/tools/pip_package:build_pip_package
    bazel-bin/tensorflow/tools/pip_package/build_pip_package --gpu "${srcdir}"/tmpoptrocm
  fi
}

_package() {
  # install headers first
  install -d "${pkgdir}"/usr/include/tensorflow
  cp -r bazel-bin/tensorflow/include/* "${pkgdir}"/usr/include/tensorflow/
  # install python-version to get all extra headers
  WHEEL_PACKAGE=$(find "${srcdir}"/$1 -name "tensor*.whl")
  pip install --ignore-installed --upgrade --root "${pkgdir}"/ $WHEEL_PACKAGE --no-dependencies
  # move extra headers to correct location
  local _srch_path="${pkgdir}/usr/lib/python$(get_pyver)"/site-packages/tensorflow/include
  check_dir "${_srch_path}"  # we need to quit on broken search paths
  find "${_srch_path}" -maxdepth 1 -mindepth 1 -type d -print0 | while read -rd $'\0' _folder; do
    cp -nr "${_folder}" "${pkgdir}"/usr/include/tensorflow/
  done
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

  # install the rest of tensorflow
  tensorflow/c/generate-pc.sh --prefix=/usr --version=${pkgver}
  sed -e 's@/include$@/include/tensorflow@' -i tensorflow.pc -i tensorflow_cc.pc
  install -Dm644 tensorflow.pc "${pkgdir}"/usr/lib/pkgconfig/tensorflow.pc
  install -Dm644 tensorflow_cc.pc "${pkgdir}"/usr/lib/pkgconfig/tensorflow_cc.pc
  install -Dm755 bazel-bin/tensorflow/libtensorflow.so "${pkgdir}"/usr/lib/libtensorflow.so.${pkgver}
  ln -s libtensorflow.so.${pkgver} "${pkgdir}"/usr/lib/libtensorflow.so.${pkgver:0:1}
  ln -s libtensorflow.so.${pkgver:0:1} "${pkgdir}"/usr/lib/libtensorflow.so
  install -Dm755 bazel-bin/tensorflow/libtensorflow_cc.so "${pkgdir}"/usr/lib/libtensorflow_cc.so.${pkgver}
  ln -s libtensorflow_cc.so.${pkgver} "${pkgdir}"/usr/lib/libtensorflow_cc.so.${pkgver:0:1}
  ln -s libtensorflow_cc.so.${pkgver:0:1} "${pkgdir}"/usr/lib/libtensorflow_cc.so
  install -Dm755 bazel-bin/tensorflow/libtensorflow_framework.so "${pkgdir}"/usr/lib/libtensorflow_framework.so.${pkgver}
  ln -s libtensorflow_framework.so.${pkgver} "${pkgdir}"/usr/lib/libtensorflow_framework.so.${pkgver:0:1}
  ln -s libtensorflow_framework.so.${pkgver:0:1} "${pkgdir}"/usr/lib/libtensorflow_framework.so
  install -Dm644 tensorflow/c/c_api.h "${pkgdir}"/usr/include/tensorflow/tensorflow/c/c_api.h
  install -Dm644 LICENSE "${pkgdir}"/usr/share/licenses/${pkgname}/LICENSE

  # Fix interoperability of C++14 and C++17. See https://bugs.archlinux.org/task/65953
  patch -Np0 -i "${srcdir}"/fix-c++17-compat.patch -d "${pkgdir}"/usr/include/tensorflow/absl/base
}

_python_package() {
  WHEEL_PACKAGE=$(find "${srcdir}"/$1 -name "tensor*.whl")
  pip install --ignore-installed --upgrade --root "${pkgdir}"/ $WHEEL_PACKAGE --no-dependencies

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
  rm -rf "${pkgdir}"/usr/bin/tensorboard

  install -Dm644 LICENSE "${pkgdir}"/usr/share/licenses/${pkgname}/LICENSE
}

package_tensorflow-rocm() {
  pkgdesc="Library for computation using data flow graphs for scalable machine learning (with ROCM)"
  depends+=(rocm rocm-libs miopen rccl)
  conflicts=(tensorflow)
  provides=(tensorflow)

  cd "${srcdir}"/tensorflow-${_pkgver}-rocm
  _package tmprocm
}

package_tensorflow-opt-rocm() {
  pkgdesc="Library for computation using data flow graphs for scalable machine learning (with ROCM and AVX2 CPU optimizations)"
  depends+=(rocm rocm-libs miopen rccl)
  conflicts=(tensorflow)
  provides=(tensorflow tensorflow-rocm)

  cd "${srcdir}"/tensorflow-${_pkgver}-opt-rocm
  _package tmpoptrocm
}

package_python-tensorflow-rocm() {
  pkgdesc="Library for computation using data flow graphs for scalable machine learning (with ROCM)"
  depends+=(tensorflow-rocm rocm rocm-libs miopen rccl "${_common_py_depends[@]}")
  conflicts=(python-tensorflow)
  provides=(python-tensorflow)

  cd "${srcdir}"/tensorflow-${_pkgver}-rocm
  _python_package tmprocm
}

package_python-tensorflow-opt-rocm() {
  pkgdesc="Library for computation using data flow graphs for scalable machine learning (with ROCM and AVX2 CPU optimizations)"
  depends+=(tensorflow-rocm rocm rocm-libs miopen rccl "${_common_py_depends[@]}")
  conflicts=(python-tensorflow)
  provides=(python-tensorflow python-tensorflow-rocm)

  cd "${srcdir}"/tensorflow-${_pkgver}-opt-rocm
  _python_package tmpoptrocm
}

# vim:set ts=2 sw=2 et:
