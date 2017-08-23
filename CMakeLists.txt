cmake_minimum_required (VERSION 2.8.11)
SET(CMAKE_CXX_STANDARD 11)
include(${CMAKE_ROOT}/Modules/ExternalProject.cmake)
project(Art3_Hoomd)
if ("${BUILD_HOST}" MATCHES "(titan-ext)[1-7]")
	set(CUDA_TOOLKIT_ROOT_DIR "$ENV{CRAY_CUDATOOLKIT_DIR}")
	set(LIBCUDA_DIR "/opt/cray/nvidia/default/lib64")
endif("${BUILD_HOST}" MATCHES "(titan-ext)[1-7]")

find_package(PythonLibs REQUIRED)
find_package(PythonInterp REQUIRED)
find_package(Boost REQUIRED)
find_package(CUDA REQUIRED)
## Setup Python Paths
include(${CMAKE_SOURCE_DIR}/PythonInstall.cmake)
configure_file (
    "${PROJECT_SOURCE_DIR}/src/wrappers/fix_cmakecache.sh.in"
	"${PROJECT_BINARY_DIR}/wrappers/fix_cmakecache.sh"
)



ExternalProject_Add(
  NUMPY_INSTALL
  DEPENDS PYTHON_2_7
  DOWNLOAD_COMMAND ""
  CONFIGURE_COMMAND ""
  BUILD_COMMAND chmod 777 ${PROJECT_BINARY_DIR}/wrappers/python_install.sh && bash ${PROJECT_BINARY_DIR}/wrappers/python_install.sh
  UPDATE_COMMAND ""
  INSTALL_COMMAND ""
)

ExternalProject_Add(
  HOOMD
  DEPENDS NUMPY_INSTALL
  GIT_REPOSITORY "https://bitbucket.org/glotzer/hoomd-blue.git"
  GIT_TAG "v1.1.1"
  CMAKE_ARGS -DBOOST_ROOT=${BOOST_ROOT} -DPYTHON_INCLUDE_DIR=${PYTHON_INCLUDE_DIR} -DCUDA_TOOLKIT_ROOT_DIR=${CUDA_TOOLKIT_ROOT_DIR} -DMPI_CXX_COMPILER=/opt/cray/craype/2.5.9/bin/CC -DMPI_C_COMPILER=/opt/cray/craype/2.5.9/bin/cc -DCUDA_ARCH_LIST=35 -DCOPY_HEADERS=On -DPYTHON_EXECUTABLE=${CMAKE_INSTALL_PREFIX}/bin/python -DNUMPY_INCLUDE_DIR=${CMAKE_INSTALL_PREFIX}/lib/python2.7/site-packages/numpy/core/include -DHOOMD_PYTHON_INCLUDE_DIR=${CMAKE_INSTALL_PREFIX}/include/python2.7 -DHOOMD_PYTHON_LIBRARY=${CMAKE_INSTALL_PREFIX}/lib64/libpython2.7.so -DCUDA_TOOLKIT_ROOT_DIR=${CUDA_TOOLKIT_ROOT_DIR} -DENABLE_CUDA=On -DCMAKE_BUILD_TYPE=RelWithDebInfo -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}/lib/python2.7 -DCUDA_USE_STATIC_CUDA_RUNTIME=Off -DCUDA_NVCC_FLAGS="RIP"
  INSTALL_DIR ${CMAKE_INSTALL_PREFIX}
  SOURCE_DIR "${PROJECT_BINARY_DIR}/3rdparty/hoomd"
  BUILD_COMMAND chmod 777 ${PROJECT_BINARY_DIR}/wrappers/fix_cmakecache.sh && ${PROJECT_BINARY_DIR}/wrappers/fix_cmakecache.sh && $(MAKE)
  UPDATE_COMMAND ""
)