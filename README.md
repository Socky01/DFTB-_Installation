# Guide to Install DFTB+

## Clone DFTB+
1. Install git  
```
sudo apt install -y git
```
2. Update and Upgrade Packages
```
sudo apt update && sudo apt upgrade -y
```
3. Cloning DFTB+ Repo
```
git clone https://github.com/dftbplus/dftbplus
```
## Configure and Build DFTB+
1. Go to dftbplus
```
cd dftbplus
```
2. Open config.cmake 
```
nano config.cmake
```
3. Edit Configuration file, our recommendation (feel free to copy and paste):
```
#
# Global architecture independent build settings
#

#set(CMAKE_BUILD_TYPE "Debug" CACHE STRING "Build type (Release|RelWithDebInfo|Debug|MinSizeRel)")
# CMAKE_BUILD_TYPE is commented out in order to allow for multi-configuration builds. It will
# automatically default to RelWithDebInfo if used in a single configuration build. Uncomment or
# override it only if you want a non-default single configuration build.

option(WITH_OMP "Whether OpenMP thread parallisation should be enabled" TRUE)

option(WITH_MPI "Whether DFTB+ should support MPI-parallelism" FALSE)
# If you build an MPI-parallised binary, consider to set WITH_OMP (OpenMP thread parallelisaton) to
# FALSE unless you want hybrid parallelisation (for experts only).

option(WITH_GPU "Whether DFTB+ should support GPU-acceleration" FALSE)
# For serial builds, the GPU support requires the MAGMA library. For MPI parallel builds it
# requires the ELSI library built with GPU support.

option(WITH_ELSI "Whether DFTB+ with MPI-parallelism should use the ELSI libraries" FALSE)
# Works only with MPI-parallel build. If WITH_GPU was selected above, the ELSI library must be
# enabled (and must have been built with GPU support).

option(WITH_TRANSPORT "Whether transport via libNEGF should be included." TRUE)
# Works only when building static libraries (see option BUILD_SHARED_LIBS)

option(WITH_POISSON "Whether the Poisson-solver should be included" ${WITH_TRANSPORT})
# The Poisson-solver is mostly used in transport calculations only. Enable this option
# if you want to use it in a non-transport build. Note, the Poisson-solver is not
# multi-instance safe and is therefore not allowed, if WITH_API (see below) is on.

option(WITH_TBLITE "Whether xTB support should be included via tblite." TRUE)

option(WITH_SOCKETS "Whether socket communication should be allowed for" FALSE)

option(WITH_ARPACK "Whether the ARPACK library should be included (needed for TD-DFTB)" TRUE)
# Works only with non-MPI (serial) build, needed for Casida linear response

option(WITH_SDFTD3 "Whether the s-dftd3 library should be included" TRUE)

option(WITH_MBD "Whether DFTB+ should be built with many-body-dispersion support" TRUE)

option(WITH_PLUMED "Whether metadynamics via the PLUMED2 library should be allowed for" FALSE)

option(WITH_CHIMES "Whether repulsive corrections via the ChIMES library should be enabled" FALSE)

option(WITH_API "Whether public API should be included and the DFTB+ library installed" TRUE)
# Turn this on, if you want to use the DFTB+ library to integrate DFTB+ into other software
# packages. (Otherwise only a stripped down version of the library without the public API is built.)
# This will also install necessary include and module files and further libraries needed to link the
# DFTB+ library.

option(WITH_PYTHON "Whether the Python components of DFTB+ should be tested and installed" TRUE)
# Use this option to test and install the Python components of DFTB+. Note, that the Python API
# based tools will only be considered, if shared library bulding (BUILD_SHARED_LIBS) and support for
# the general API (WITH_API) and dynamic loading (ENABLE_DYNAMIC_LOADING) had been enabled.
# Otherwise only the file I/O based tools (dptools) will be tested and installed.

option(INSTANCE_SAFE_BUILD "Whether build should support concurrent DFTB+ instances" FALSE)
# Turn this on, if you want to create multiple concurrent DFTB+ instances **within one process** via
# the API. This option will ensure that only components without writable global variables are
# included in the build, so that multiple instances can safely coexist. There are components
# (e.g. Poisson, DFTD-D3, ARPACK) which can not be included if this option is on. Note, this option
# is not relevant for the standalone DFTB+ binary, only for the API (if WITH_API had been turned
# on).

option(BUILD_SHARED_LIBS "Whether the libraries built should be shared" FALSE)
# Turn this on, if the DFTB+ library (and other compiled libraries) should be shared libraries and
# dynamically linked to their applications. This results in smaller applications, but the libraries
# must be present at run-time (and the correct LD_LIBRARY_PATH environment variable must be set, so
# that they can be found by the operating system). If you want use the DFTB+ library from other
# software packages (see WITH_API option above), they may also require a shared library (e.g.
# calling DFTB+ functions from Python or Julia). Note, that in order to use the library from Python
# and Julia, you also need to turn on the ENABLE_DYNAMIC_LOADING option.

option(ENABLE_DYNAMIC_LOADING "Whether the library should be dynamically loadable" FALSE)
# Turn this on, if you wish to load the library dynamically (typically when you want to use
# the library from Python or Julia). Only makes sense in combination with BUILD_SHARED_LIBS and
# WITH_API set to True.

#
# Test environment settings
#
set(TEST_MPI_PROCS "1" CACHE STRING "Nr. of MPI processes used for testing")

set(TEST_OMP_THREADS "1" CACHE STRING "Nr. of OpenMP-threads used for testing")

# Command line used to launch the test code.
# The escaped variables (\${VARIABLE}) will be substituted by the corresponding CMake variables.
if(WITH_MPI)
  set(TEST_RUNNER_TEMPLATE "env OMP_NUM_THREADS=\${TEST_OMP_THREADS} mpiexec -n \${TEST_MPI_PROCS}"
    CACHE STRING "How to run the tests")
else()
  set(TEST_RUNNER_TEMPLATE "env OMP_NUM_THREADS=\${TEST_OMP_THREADS}" CACHE STRING
    "How to run the tests")
  set(MODES_RUNNER_TEMPLATE "env OMP_NUM_THREADS=\${TEST_OMP_THREADS}" CACHE STRING
    "How to run the modes code for tests")
endif()

# Turn it on to include the unit tests (needes FyTest)
option(WITH_UNIT_TESTS "Whether the unit tests should be built" FALSE)

#
# Installation options
#
set(CMAKE_INSTALL_PREFIX "${CMAKE_BINARY_DIR}/_install" CACHE STRING
  "Directory to install the compiled code into")

set(INSTALL_INCLUDEDIR "dftbplus" CACHE PATH
  "Name of the project specific sub-folder within the install folder for include files")

set(INSTALL_MODULEDIR "${INSTALL_INCLUDEDIR}/modfiles" CACHE PATH
  "Installation directory for Fortran module files (within the install folder for include files)")

set(PKGCONFIG_LANGUAGE "Fortran" CACHE STRING
  "Compiler and Linker language to assume when creating the pkg-config export file (C or Fortran)")
# The pkg-config export file (lib/pkgconfig/dftbplus.pc) contains the compiler and linker options
# needed to link the DFTB+ library to an application. (It can be queried with the pkg-config tool.)
# Depending on the language setting ("C" or "Fortran") you would get the flags for the case of using
# that compiler for the linking.


#
# Advanced options (e.g. for developers and packagers)
#

#set(TOOLCHAIN "gnu" CACHE STRING "Prefix of the toolchain file to be read from the sys/ folder")
# Uncomment and set it if you want to override the automatic, compiler based toolchain file
# selection.

set(HYBRID_CONFIG_METHODS "Submodule;Find;Fetch" CACHE STRING
  "Configuration methods to try in order to satisfy hybrid dependencies")
#
# This list can be used to control how hybrid dependencies (external dependencies which can
# optionally be built during the build process) are configured. The listed methods are applied in
# the specified order. Following methods are available:
#
# Submodule: Use the source in external/*/origin and build the dependency as part of the build
#     process. If the source is not present, try to retrieve it via the 'git submodule' command
#     (provided the source tree is a git repository and git is available)
#
# Find: Find the dependency as an already installed package in the system.
#
# Fetch: Fetch the source into the build folder and build the dependency as part of the build
#     process (works also in cases where the source tree is not a Git repository)

#
# Developer settings
#
option(LCOV_REPORT "Whether coverage report should be generated via lcov/genhtml" FALSE)
# Makes only sense for build type 'Coverage'. Requires lcov and and optionally genhtml to be
# installed on the system. After building the code, you have to build manually the 'lcov_init'
# target (e.g. `make lcov_init`), then run the tests (e.g. `ctest`) and finally generate the report
# with the lcov_report target (e.g. `make lcov_report`). If you only need the evaluated coverage
# data, but no HTML report, build the `lcov_eval` target instead.
```
4. Save and Exit
```
ctrl + o
enter
ctrl + x
```
6. Move dftbplus directory to new directory called opt
```
cd ..
```
```
mkdir opt
```
```
mv dftbplus/ opt/
```
```
cd opt
```
```
cd dftbplus
```
7. Installing required library
```
sudo apt install -y libblas-dev libarpack2-dev
```
8. Configure Cmake enviroment FC=gfortran dan CC=GCC. Required packages: cmake, gfortran and gcc.
```
sudo apt install -y cmake gfortran gcc
```
```
FC=gfortran CC=gcc cmake -DCMAKE_INSTALL_PREFIX=$HOME/$USER/dftb+ -B build .
```
9. If there's an error remove build directory and retry setting enviroment. __IF THERE IS NO ERROR GO TO STEP 10__
```
rm -rf build
```
```
FC=gfortran CC=gcc cmake -DCMAKE_INSTALL_PREFIX=$HOME/$USER/dftb+ -B build .
```
10. Build Cmake Build
```
cmake --build build -- -j 10
```
__10 is number of CPUs in, we will use 10 CPUs.
To check number of CPUs available use command:__
```
lscpu
```

## Download Required Parameters for DFTB+
1.  Download ALL paramters
```
./utils/get_opt_externals ALL
```
2. Test dftbplus
```
cd build
```
```
ctest -j 10
``` 
__10 is number of CPUs in, we will use 10 CPUs.
To check number of CPUs available use command:__
```
lscpu
```
3. Install build: 
```
cd ..
```
```
cmake --install build
```

## Aliasing DFTB+
1. Open bashrc
```
nano ~/.bashrc 
```
2. Paste these comand in the bottom line of .bashrc:
```
alias dftb+="home/$USER/opt/dftbplus/build/app/dftb+/dftb+"
alias modes+="home/$USER/opt/dftbplus/build/app/modes/modes"
alias phonons="home/$USER/opt/dftbplus/build/app/phonons/phonons"
alias waveplot="home/$USER/opt/dftbplus/build/app/waveplot/waveplot"
```
3. Save and Exit
```
ctrl + o
enter
ctrl + x
```
4. Refresh bashrc
```
source ~/.bashrc
```
5. After aliasing we can call these program using command:
DFTB+ ``` dftb+ ```
Modes ``` modes ```
Phonons ``` phonons ```
Waveplot ``` waveplot ```
