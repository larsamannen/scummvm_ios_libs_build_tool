cmake_minimum_required(VERSION 3.21)

project(
  scummvm-ios-libs-v4
  VERSION 4.0
  LANGUAGES C CXX)

# Avoid warning about DOWNLOAD_EXTRACT_TIMESTAMP in CMake 3.24:
if (CMAKE_VERSION VERSION_GREATER_EQUAL "3.24.0")
 	cmake_policy(SET CMP0135 NEW)
endif()

set(CMAKE_VERBOSE_MAKEFILE ON)

include(FetchContent)
include(ExternalProject)

find_program(XCRUN_EXECUTABLE xcrun)
if(NOT XCRUN_EXECUTABLE)
  message(FATAL_ERROR "xcrun not found. Please install either the standalone commandline tools or Xcode.")
endif()

set(SDK_NAME iphoneos)

execute_process(COMMAND ${XCRUN_EXECUTABLE} --sdk ${SDK_NAME} --show-sdk-path
  OUTPUT_VARIABLE CMAKE_OSX_SYSROOT_INT
  ERROR_QUIET
  OUTPUT_STRIP_TRAILING_WHITESPACE)

set(PREFIX ${CMAKE_CURRENT_SOURCE_DIR}/libs)
set(DEFAULT_FLAGS "${CMAKE_C_FLAGS} -isysroot ${CMAKE_OSX_SYSROOT_INT} -miphoneos-version-min=7.0 -arch arm64 -arch armv7 -arch armv7s -I${PREFIX}/include -L${PREFIX}/lib")
set(PKG_CONFIG_PATH ${PREFIX}/lib/pkgconfig/)

#bzip2
FetchContent_Declare(_bzip2
  URL https://sourceware.org/pub/bzip2/bzip2-1.0.8.tar.gz
  URL_HASH MD5=67e051268d0c475ea773822f7500d0e5
  DOWNLOAD_DIR ${CMAKE_CURRENT_BINARY_DIR}
  DOWNLOAD_NO_EXTRACT TRUE
  TLS_VERIFY TRUE
)
FetchContent_MakeAvailable(_bzip2)
file(ARCHIVE_EXTRACT INPUT ${CMAKE_CURRENT_BINARY_DIR}/bzip2-1.0.8.tar.gz)

ExternalProject_Add(bzip2
  SOURCE_DIR ${CMAKE_CURRENT_BINARY_DIR}/bzip2-1.0.8
  INSTALL_DIR ${PREFIX}
  CONFIGURE_COMMAND ""
  BUILD_COMMAND ${CMAKE_COMMAND} -E env
  make libbz2.a CFLAGS=${DEFAULT_FLAGS} "-Wall -Winline -O2 -D_FILE_OFFSET_BITS=64" LDFLAGS=${DEFAULT_FLAGS}
  BUILD_IN_SOURCE TRUE
  INSTALL_COMMAND mkdir <INSTALL_DIR>/lib && cp -f libbz2.a <INSTALL_DIR>/lib && mkdir <INSTALL_DIR>/include && cp -f bzlib.h <INSTALL_DIR>/include
)

#libpng16
FetchContent_Declare(_libpng16
  URL https://download.sourceforge.net/libpng/libpng-1.6.39.tar.gz
  URL_HASH MD5=93b8e79a008747e70f7704f600349559
  DOWNLOAD_DIR ${CMAKE_CURRENT_BINARY_DIR}
  DOWNLOAD_NO_EXTRACT TRUE
  TLS_VERIFY TRUE
)
FetchContent_MakeAvailable(_libpng16)
file(ARCHIVE_EXTRACT INPUT ${CMAKE_CURRENT_BINARY_DIR}/libpng-1.6.39.tar.gz)

ExternalProject_Add(libpng16
  SOURCE_DIR ${CMAKE_CURRENT_BINARY_DIR}/libpng-1.6.39
  INSTALL_DIR ${PREFIX}
  CONFIGURE_COMMAND ./configure "--host=arm-apple-darwin" "--prefix=" CFLAGS=${DEFAULT_FLAGS} LDFLAGS=${DEFAULT_FLAGS}
  BUILD_COMMAND ${CMAKE_COMMAND} -E env
  make
  BUILD_IN_SOURCE TRUE
  INSTALL_COMMAND make DESTDIR=<INSTALL_DIR> install-libLTLIBRARIES install-binSCRIPTS install-pkgconfigDATA install-pkgincludeHEADERS install-nodist_pkgincludeHEADERS install-header-links install-library-links install-libpng-pc install-libpng-config
)

#libjpeg-turbo
FetchContent_Declare(_libjpeg-turbo
  URL https://sourceforge.net/projects/libjpeg-turbo/files/2.0.6/libjpeg-turbo-2.0.6.tar.gz
  URL_HASH MD5=4cada3f0bdc93d826fa31bf9e4469ef6
  DOWNLOAD_DIR ${CMAKE_CURRENT_BINARY_DIR}
  DOWNLOAD_NO_EXTRACT TRUE
  TLS_VERIFY TRUE
)
FetchContent_MakeAvailable(_libjpeg-turbo)
file(ARCHIVE_EXTRACT INPUT ${CMAKE_CURRENT_BINARY_DIR}/libjpeg-turbo-2.0.6.tar.gz)

ExternalProject_Add(libjpeg-turbo
  SOURCE_DIR ${CMAKE_CURRENT_BINARY_DIR}/libjpeg-turbo-2.0.6
  INSTALL_DIR ${PREFIX}
  CMAKE_ARGS
    -DENABLE_SHARED=0
    -DWITH_TURBOJPEG=0
    -DREQUIRE_SIMD=0
    -DCMAKE_C_FLAGS=${DEFAULT_FLAGS}
    -DCMAKE_TOOLCHAIN_FILE=${CMAKE_CURRENT_SOURCE_DIR}/ios.toolchain.cmake
  INSTALL_COMMAND make DESTDIR=<INSTALL_DIR> install
)

#giflib
FetchContent_Declare(_giflib
  URL https://sourceforge.net/projects/giflib/files/giflib-5.1.9.tar.gz
  URL_HASH MD5=179336e739eeacee4cef2a8f789a0fcb
  DOWNLOAD_DIR ${CMAKE_CURRENT_BINARY_DIR}
  DOWNLOAD_NO_EXTRACT TRUE
  TLS_VERIFY TRUE
)
FetchContent_MakeAvailable(_giflib)
file(ARCHIVE_EXTRACT INPUT ${CMAKE_CURRENT_BINARY_DIR}/giflib-5.1.9.tar.gz)

ExternalProject_Add(giflib
  SOURCE_DIR ${CMAKE_CURRENT_BINARY_DIR}/giflib-5.1.9
  INSTALL_DIR ${PREFIX}
  CONFIGURE_COMMAND ""
  BUILD_COMMAND ${CMAKE_COMMAND} -E env
  make libgif.a OFLAGS=${DEFAULT_FLAGS}
  BUILD_IN_SOURCE TRUE
  INSTALL_COMMAND make DESTDIR=<INSTALL_DIR>/lib install-include && cp -f libgif.a <INSTALL_DIR>/lib
)

#faad2
find_program(AUTORECONF_EXECUTABLE autoreconf REQUIRED)
if(NOT AUTORECONF_EXECUTABLE)
  message(FATAL_ERROR "Install autoreconf from brew: brew install automake")
endif()

ExternalProject_Add(faad2
  GIT_REPOSITORY https://github.com/knik0/faad2.git
  GIT_SHALLOW TRUE
  GIT_TAG 2_10_0
  INSTALL_DIR ${PREFIX}
  CONFIGURE_COMMAND autoreconf -fi && ./configure "--host=arm-apple-darwin" "--prefix=" CFLAGS=${DEFAULT_FLAGS} LDFLAGS=${DEFAULT_FLAGS}
  BUILD_COMMAND ${CMAKE_COMMAND} -E env
  make -C libfaad
  BUILD_IN_SOURCE TRUE
  INSTALL_COMMAND make -C libfaad DESTDIR=<INSTALL_DIR>/lib install
)

#libmad
FetchContent_Declare(_libmad
  URL https://downloads.sourceforge.net/mad/libmad-0.15.1b.tar.gz
  URL_HASH MD5=1be543bc30c56fb6bea1d7bf6a64e66c
  DOWNLOAD_DIR ${CMAKE_CURRENT_BINARY_DIR}
  DOWNLOAD_NO_EXTRACT TRUE
  TLS_VERIFY TRUE
)
FetchContent_MakeAvailable(_libmad)
file(ARCHIVE_EXTRACT INPUT ${CMAKE_CURRENT_BINARY_DIR}/libmad-0.15.1b.tar.gz)

ExternalProject_Add(libmad
  SOURCE_DIR ${CMAKE_CURRENT_BINARY_DIR}/libmad-0.15.1b
  INSTALL_DIR ${PREFIX}
  PATCH_COMMAND ${CMAKE_COMMAND} -E copy_if_different ${CMAKE_CURRENT_SOURCE_DIR}/patches/libmad-0.15.1b-fixes-1.patch <SOURCE_DIR> && patch -p1 < <SOURCE_DIR>/libmad-0.15.1b-fixes-1.patch
  CONFIGURE_COMMAND touch NEWS AUTHORS ChangeLog && autoreconf -fi && ./configure "--host=arm64-apple-darwin" "--prefix=" "ac_cv_func_fork=no" CFLAGS=${DEFAULT_FLAGS} LDFLAGS=${DEFAULT_FLAGS}
  BUILD_COMMAND ${CMAKE_COMMAND} -E env
  make
  BUILD_IN_SOURCE TRUE
  INSTALL_COMMAND make DESTDIR=<INSTALL_DIR> install
)

#libogg
FetchContent_Declare(_libogg
  URL https://downloads.xiph.org/releases/ogg/libogg-1.3.5.tar.xz
  URL_HASH MD5=3178c98341559657a15b185bf5d700a5
  DOWNLOAD_DIR ${CMAKE_CURRENT_BINARY_DIR}
  DOWNLOAD_NO_EXTRACT TRUE
  TLS_VERIFY TRUE
)
FetchContent_MakeAvailable(_libogg)
file(ARCHIVE_EXTRACT INPUT ${CMAKE_CURRENT_BINARY_DIR}/libogg-1.3.5.tar.xz)

ExternalProject_Add(libogg
  SOURCE_DIR ${CMAKE_CURRENT_BINARY_DIR}/libogg-1.3.5
  INSTALL_DIR ${PREFIX}
  CONFIGURE_COMMAND autoreconf -fi && ./configure "--host=arm64-apple-darwin" "--prefix=" CFLAGS=${DEFAULT_FLAGS} LDFLAGS=${DEFAULT_FLAGS}
  BUILD_COMMAND ${CMAKE_COMMAND} -E env
  make
  BUILD_IN_SOURCE TRUE
  INSTALL_COMMAND make DESTDIR=<INSTALL_DIR> install
)

#libtheora
FetchContent_Declare(_libtheora
  URL http://downloads.xiph.org/releases/theora/libtheora-1.1.1.tar.bz2
  URL_HASH MD5=292ab65cedd5021d6b7ddd117e07cd8e
  DOWNLOAD_DIR ${CMAKE_CURRENT_BINARY_DIR}
  DOWNLOAD_NO_EXTRACT TRUE
  TLS_VERIFY TRUE
)
FetchContent_MakeAvailable(_libtheora)
file(ARCHIVE_EXTRACT INPUT ${CMAKE_CURRENT_BINARY_DIR}/libtheora-1.1.1.tar.bz2)

ExternalProject_Add(libtheora
  DEPENDS libogg
  SOURCE_DIR ${CMAKE_CURRENT_BINARY_DIR}/libtheora-1.1.1
  INSTALL_DIR ${PREFIX}
  CONFIGURE_COMMAND autoreconf -fi -I m4 && PKG_CONFIG_PATH=${PKG_CONFIG_PATH} ./configure "--host=arm64-apple-darwin" "--prefix=" "--disable-examples" "--disable-spec" "--disable-doc" CFLAGS=${DEFAULT_FLAGS}  LDFLAGS=${DEFAULT_FLAGS}
  BUILD_COMMAND ${CMAKE_COMMAND} -E env
  make
  BUILD_IN_SOURCE TRUE
  INSTALL_COMMAND make DESTDIR=<INSTALL_DIR> install
)

#libvorbis
FetchContent_Declare(_libvorbis
  URL https://downloads.xiph.org/releases/vorbis/libvorbis-1.3.7.tar.xz
  URL_HASH MD5=50902641d358135f06a8392e61c9ac77
  DOWNLOAD_DIR ${CMAKE_CURRENT_BINARY_DIR}
  DOWNLOAD_NO_EXTRACT TRUE
  TLS_VERIFY TRUE
)
FetchContent_MakeAvailable(_libvorbis)
file(ARCHIVE_EXTRACT INPUT ${CMAKE_CURRENT_BINARY_DIR}/libvorbis-1.3.7.tar.xz)

ExternalProject_Add(libvorbis
  DEPENDS libogg
  SOURCE_DIR ${CMAKE_CURRENT_BINARY_DIR}/libvorbis-1.3.7
  INSTALL_DIR ${PREFIX}
  CONFIGURE_COMMAND autoreconf -fi -I m4 && PKG_CONFIG_PATH=${PKG_CONFIG_PATH} ./configure "--host=arm64-apple-darwin" "--prefix=" CFLAGS=${DEFAULT_FLAGS}  LDFLAGS=${DEFAULT_FLAGS}
  BUILD_COMMAND ${CMAKE_COMMAND} -E env
  make
  BUILD_IN_SOURCE TRUE
  INSTALL_COMMAND make DESTDIR=<INSTALL_DIR> install
)

#libvorbis
FetchContent_Declare(_flac
  URL https://downloads.xiph.org/releases/flac/flac-1.4.2.tar.xz
  URL_HASH MD5=ca9140f37b286d2571e37d66aae50f92
  DOWNLOAD_DIR ${CMAKE_CURRENT_BINARY_DIR}
  DOWNLOAD_NO_EXTRACT TRUE
  TLS_VERIFY TRUE
)
FetchContent_MakeAvailable(_flac)
file(ARCHIVE_EXTRACT INPUT ${CMAKE_CURRENT_BINARY_DIR}/flac-1.4.2.tar.xz)

ExternalProject_Add(flac
  SOURCE_DIR ${CMAKE_CURRENT_BINARY_DIR}/flac-1.4.2
  INSTALL_DIR ${PREFIX}
  CONFIGURE_COMMAND autoreconf -fi && ./configure "--host=arm-apple-darwin" "--prefix=" "--disable-shared" "--enable-static" "--disable-doxygen-docs" "--disable-xmms-plugin" "--disable-cpplibs" "--disable-ogg" CFLAGS=${DEFAULT_FLAGS} LDFLAGS=${DEFAULT_FLAGS}
  BUILD_COMMAND ${CMAKE_COMMAND} -E env
  make -C src/libFLAC
  BUILD_IN_SOURCE TRUE
  INSTALL_COMMAND make -C src/libFLAC DESTDIR=<INSTALL_DIR> install && make -C include DESTDIR=<INSTALL_DIR> install
)

#libmpeg2
FetchContent_Declare(_libmpeg2
  URL https://libmpeg2.sourceforge.net/files/libmpeg2-0.5.1.tar.gz
  URL_HASH MD5=0f92c7454e58379b4a5a378485bbd8ef
  DOWNLOAD_DIR ${CMAKE_CURRENT_BINARY_DIR}
  DOWNLOAD_NO_EXTRACT TRUE
  TLS_VERIFY TRUE
)
FetchContent_MakeAvailable(_libmpeg2)
file(ARCHIVE_EXTRACT INPUT ${CMAKE_CURRENT_BINARY_DIR}/libmpeg2-0.5.1.tar.gz)

ExternalProject_Add(libmpeg2
  SOURCE_DIR ${CMAKE_CURRENT_BINARY_DIR}/libmpeg2-0.5.1
  INSTALL_DIR ${PREFIX}
  CONFIGURE_COMMAND autoreconf -fi && ./configure "--host=arm64-apple-darwin" "--prefix=" "--disable-shared" "--enable-static" CFLAGS=${DEFAULT_FLAGS} LDFLAGS=${DEFAULT_FLAGS}
  BUILD_COMMAND ${CMAKE_COMMAND} -E env
  make -C libmpeg2
  BUILD_IN_SOURCE TRUE
  INSTALL_COMMAND make -C libmpeg2 DESTDIR=<INSTALL_DIR> install && make -C include DESTDIR=<INSTALL_DIR> install
)

#liba52dec
FetchContent_Declare(_liba52dec
  URL https://liba52.sourceforge.net/files/a52dec-0.7.4.tar.gz
  URL_HASH MD5=caa9f5bc44232dc8aeea773fea56be80
  DOWNLOAD_DIR ${CMAKE_CURRENT_BINARY_DIR}
  DOWNLOAD_NO_EXTRACT TRUE
  TLS_VERIFY TRUE
)
FetchContent_MakeAvailable(_liba52dec)
file(ARCHIVE_EXTRACT INPUT ${CMAKE_CURRENT_BINARY_DIR}/a52dec-0.7.4.tar.gz)

ExternalProject_Add(liba52dec
  SOURCE_DIR ${CMAKE_CURRENT_BINARY_DIR}/a52dec-0.7.4
  INSTALL_DIR ${PREFIX}
  CONFIGURE_COMMAND autoreconf -fi && ./configure "--host=arm64-apple-darwin" "--prefix=" "--disable-shared" "--enable-static" CFLAGS=${DEFAULT_FLAGS} LDFLAGS=${DEFAULT_FLAGS}
  BUILD_COMMAND ${CMAKE_COMMAND} -E env
  make -C liba52
  BUILD_IN_SOURCE TRUE
  INSTALL_COMMAND make -C liba52 DESTDIR=<INSTALL_DIR> install && make -C include DESTDIR=<INSTALL_DIR> install
)

#libvpx
FetchContent_Declare(_libvpx
  URL https://github.com/webmproject/libvpx/archive/v1.13.0/libvpx-1.13.0.tar.gz
  URL_HASH MD5=d5fd45a806a65a57d6635f9e7a98a1b2
  DOWNLOAD_DIR ${CMAKE_CURRENT_BINARY_DIR}
  DOWNLOAD_NO_EXTRACT TRUE
  TLS_VERIFY TRUE
)
FetchContent_MakeAvailable(_libvpx)
file(ARCHIVE_EXTRACT INPUT ${CMAKE_CURRENT_BINARY_DIR}/libvpx-1.13.0.tar.gz)

set(TARGETS "arm64-darwin-gcc" "armv7-darwin-gcc" "armv7s-darwin-gcc")

foreach(TARGET IN LISTS TARGETS)
  ExternalProject_Add(libvpx${TARGET}
    SOURCE_DIR ${CMAKE_CURRENT_BINARY_DIR}/libvpx-1.13.0
    INSTALL_DIR ${PREFIX}
    CONFIGURE_COMMAND prefix="" ./configure "--target=${TARGET}" "--disable-shared" "--enable-static" "--disable-examples" "--disable-tools" "--disable-docs" "--disable-unit-tests" "--disable-install-bins" "--disable-install-srcs" "--size-limit=16384x16384" "--disable-vp8-encoder" "--disable-vp9-encoder"
    BUILD_COMMAND ${CMAKE_COMMAND} -E env
    make clean && make
    BUILD_IN_SOURCE TRUE
    INSTALL_COMMAND make DESTDIR=<INSTALL_DIR>/${TARGET} install
  )
endforeach()

find_program (LIPO lipo REQUIRED)

# Create a fat library. This will assume that libs/lib folder exist so it has to be crated
add_custom_target(libvpx
  DEPENDS libvpxarm64-darwin-gcc libvpxarmv7-darwin-gcc libvpxarmv7s-darwin-gcc
  COMMAND ${LIPO} ${PREFIX}/arm64-darwin-gcc/lib/libvpx.a ${PREFIX}/armv7-darwin-gcc/lib/libvpx.a ${PREFIX}/armv7s-darwin-gcc/lib/libvpx.a -create -output ${PREFIX}/lib/libvpx.a
)
