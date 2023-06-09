cmake_minimum_required(VERSION 3.21)

project(
  scummvm-ios-libs-v4
  VERSION 4.0
  LANGUAGES C CXX)

# Avoid warning about DOWNLOAD_EXTRACT_TIMESTAMP in CMake 3.24:
if (CMAKE_VERSION VERSION_GREATER_EQUAL "3.24.0")
 	cmake_policy(SET CMP0135 NEW)
endif()

include(FetchContent)
include(ExternalProject)
set(FETCHCONTENT_QUIET FALSE)

find_program(XCRUN_EXECUTABLE xcrun)
if(NOT XCRUN_EXECUTABLE)
  message(FATAL_ERROR "xcrun not found. Please install either the standalone commandline tools or Xcode.")
endif()

set(SDK_NAME iphonesimulator)

execute_process(COMMAND ${XCRUN_EXECUTABLE} --sdk ${SDK_NAME} --show-sdk-path
  OUTPUT_VARIABLE CMAKE_OSX_SYSROOT_INT
  ERROR_QUIET
  OUTPUT_STRIP_TRAILING_WHITESPACE)

set(PREFIX ${CMAKE_CURRENT_SOURCE_DIR}/libs)
set(DEFAULT_FLAGS "${CMAKE_C_FLAGS} -isysroot ${CMAKE_OSX_SYSROOT_INT} -mios-simulator-version-min=7.0 -arch arm64 -I${PREFIX}/include -L${PREFIX}/lib")
set(DEFAULT_arm64_GLIB_FLAGS "${CMAKE_C_FLAGS} -isysroot ${CMAKE_OSX_SYSROOT_INT} -mios-simulator-version-min=7.0 -arch arm64 -I${PREFIX}/include -L${PREFIX}/lib -framework CoreFoundation")
set(DEFAULT_64_FLAGS "${CMAKE_C_FLAGS} -isysroot ${CMAKE_OSX_SYSROOT_INT} -mios-simulator-version-min=7.0 -arch arm64 -I${PREFIX}/include -L${PREFIX}/lib")
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

set(TARGETS "arm64-darwin-gcc")

foreach(TARGET IN LISTS TARGETS)
  ExternalProject_Add(libvpx${TARGET}
    SOURCE_DIR ${CMAKE_CURRENT_BINARY_DIR}/libvpx-1.13.0
    INSTALL_DIR ${PREFIX}
    CONFIGURE_COMMAND ${CMAKE_COMMAND} -E env
    prefix="" ./configure "--target=x86_64-iphonesimulator-gcc" "--disable-shared" "--enable-static" "--disable-examples" "--disable-tools" "--disable-docs" "--disable-unit-tests" "--disable-install-bins" "--disable-install-srcs" "--size-limit=16384x16384" "--disable-vp8-encoder" "--disable-vp9-encoder"
    BUILD_COMMAND ${CMAKE_COMMAND} -E env
    make clean && make
    BUILD_IN_SOURCE TRUE
    INSTALL_COMMAND make DESTDIR=<INSTALL_DIR>/${TARGET} install
  )
endforeach()

#find_program (LIPO lipo REQUIRED)

# Create a fat library. This will assume that libs/lib folder exist so it has to be crated
add_custom_target(libvpx
  DEPENDS libvpxarm64-darwin-gcc
#  COMMAND ${LIPO} ${PREFIX}/arm64-darwin-gcc/lib/libvpx.a ${PREFIX}/armv7-darwin-gcc/lib/l#ibvpx.a ${PREFIX}/armv7s-darwin-gcc/lib/libvpx.a -create -output ${PREFIX}/lib/libvpx.a
)

#libcurl
FetchContent_Declare(_libcurl
  URL https://curl.se/download/curl-7.74.0.tar.xz
  URL_HASH MD5=448156d9cd05009b0097b43de639fd34
  DOWNLOAD_DIR ${CMAKE_CURRENT_BINARY_DIR}
  DOWNLOAD_NO_EXTRACT TRUE
  TLS_VERIFY TRUE
)
FetchContent_MakeAvailable(_libcurl)
file(ARCHIVE_EXTRACT INPUT ${CMAKE_CURRENT_BINARY_DIR}/curl-7.74.0.tar.xz)

set(CPUS "64")

foreach(CPU IN LISTS CPUS)
  ExternalProject_Add(libcurl${CPU}
    SOURCE_DIR ${CMAKE_CURRENT_BINARY_DIR}/curl-7.74.0
    INSTALL_DIR ${PREFIX}
    CONFIGURE_COMMAND autoreconf -fi -I m4 && ./configure --host=arm-apple-darwin --disable-shared --enable-static --with-secure-transport --disable-ntlm-wb --without-zstd --prefix= CFLAGS=${DEFAULT_${CPU}_FLAGS} LDFLAGS=${DEFAULT_${CPU}_FLAGS}
    BUILD_COMMAND ${CMAKE_COMMAND} -E env
    make clean && make -C lib
    BUILD_IN_SOURCE TRUE
    INSTALL_COMMAND make DESTDIR=<INSTALL_DIR>/${CPU} -C lib install && make DESTDIR=<INSTALL_DIR> -C include install
  )
endforeach()

# Create a fat library. This will assume that libs/lib folder exist so it has to be crated
add_custom_target(libcurl
  DEPENDS  libcurl64
#  COMMAND ${LIPO} ${PREFIX}/32/lib/libcurl.a ${PREFIX}/64/lib/libcurl.a -create -output ${PREFIX}/lib/libcurl.a
)

#freetype
FetchContent_Declare(_freetype
  URL https://downloads.sourceforge.net/freetype/freetype-2.13.0.tar.xz
  URL_HASH MD5=fe73327ebbf0b629f3ad24be656d59de
  DOWNLOAD_DIR ${CMAKE_CURRENT_BINARY_DIR}
  DOWNLOAD_NO_EXTRACT TRUE
  TLS_VERIFY TRUE
)
FetchContent_MakeAvailable(_freetype)
file(ARCHIVE_EXTRACT INPUT ${CMAKE_CURRENT_BINARY_DIR}/freetype-2.13.0.tar.xz)

ExternalProject_Add(freetype
  SOURCE_DIR ${CMAKE_CURRENT_BINARY_DIR}/freetype-2.13.0
  INSTALL_DIR ${PREFIX}
  CONFIGURE_COMMAND ./configure "--host=arm64-apple-darwin" "--prefix=" "--disable-shared" "--enable-static" "--with-harfbuzz=no" CFLAGS=${DEFAULT_FLAGS} LDFLAGS=${DEFAULT_FLAGS}
  BUILD_COMMAND ${CMAKE_COMMAND} -E env
  make
  BUILD_IN_SOURCE TRUE
  INSTALL_COMMAND make DESTDIR=<INSTALL_DIR> install
)

#fribidi
FetchContent_Declare(_fribidi
  URL https://github.com/fribidi/fribidi/releases/download/v1.0.12/fribidi-1.0.12.tar.xz
  URL_HASH MD5=21185b398635a7fc0d3ff0a7578c4791
  DOWNLOAD_DIR ${CMAKE_CURRENT_BINARY_DIR}
  DOWNLOAD_NO_EXTRACT TRUE
  TLS_VERIFY TRUE
)
FetchContent_MakeAvailable(_fribidi)
file(ARCHIVE_EXTRACT INPUT ${CMAKE_CURRENT_BINARY_DIR}/fribidi-1.0.12.tar.xz)

ExternalProject_Add(fribidi
  SOURCE_DIR ${CMAKE_CURRENT_BINARY_DIR}/fribidi-1.0.12
  INSTALL_DIR ${PREFIX}
  CONFIGURE_COMMAND NOCONFIGURE=1 ./autogen.sh && ./configure "--host=arm64-apple-darwin" "--prefix=" "--disable-shared" "--enable-static" CFLAGS=${DEFAULT_FLAGS} LDFLAGS=${DEFAULT_FLAGS}
  BUILD_COMMAND ${CMAKE_COMMAND} -E env
  make
  BUILD_IN_SOURCE TRUE
  INSTALL_COMMAND make DESTDIR=<INSTALL_DIR> install
)

#libsdl2-net
ExternalProject_Add(libsdl2-net
  GIT_REPOSITORY https://github.com/libsdl-org/SDL_net.git
  GIT_SHALLOW TRUE
  GIT_TAG release-2.0.1
  INSTALL_DIR ${PREFIX}
  PATCH_COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_CURRENT_SOURCE_DIR}/patches/no-sdl.patch ${CMAKE_CURRENT_SOURCE_DIR}/patches/stub-showinterfaces.patch  <SOURCE_DIR> && patch -p1 < <SOURCE_DIR>/no-sdl.patch && patch -p1 < <SOURCE_DIR>/stub-showinterfaces.patch
  CONFIGURE_COMMAND ./autogen.sh && ./configure "--host=arm-apple-darwin" "--prefix=" CFLAGS=${DEFAULT_FLAGS} LDFLAGS=${DEFAULT_FLAGS}
  BUILD_COMMAND ${CMAKE_COMMAND} -E env
  make
  BUILD_IN_SOURCE TRUE
  INSTALL_COMMAND make  DESTDIR=<INSTALL_DIR> install
)

#libffi
FetchContent_Declare(_libffi
  URL https://github.com/libffi/libffi/releases/download/v3.4.4/libffi-3.4.4.tar.gz
  URL_HASH MD5=0da1a5ed7786ac12dcbaf0d499d8a049
  DOWNLOAD_DIR ${CMAKE_CURRENT_BINARY_DIR}
  DOWNLOAD_NO_EXTRACT TRUE
  TLS_VERIFY TRUE
)
FetchContent_MakeAvailable(_libffi)
file(ARCHIVE_EXTRACT INPUT ${CMAKE_CURRENT_BINARY_DIR}/libffi-3.4.4.tar.gz)

ExternalProject_Add(libffi
  SOURCE_DIR ${CMAKE_CURRENT_BINARY_DIR}/libffi-3.4.4
  INSTALL_DIR ${PREFIX}
  CONFIGURE_COMMAND ./configure "--host=arm64-apple-darwin" "--prefix=" "--disable-shared" "--enable-static" "--disable-builddir" CFLAGS=${DEFAULT_FLAGS} LDFLAGS=${DEFAULT_FLAGS}
  BUILD_COMMAND ${CMAKE_COMMAND} -E env
  make
  BUILD_IN_SOURCE TRUE
  INSTALL_COMMAND make -C include DESTDIR=<INSTALL_DIR> install && make DESTDIR=<INSTALL_DIR> install-pkgconfigDATA install-toolexeclibLTLIBRARIES
)

#gettext
FetchContent_Declare(_gettext
  URL https://ftp.gnu.org/pub/gnu/gettext/gettext-0.21.1.tar.gz
  URL_HASH MD5=83fb74df9fac4d073497cecb0fe17e8b
  DOWNLOAD_DIR ${CMAKE_CURRENT_BINARY_DIR}
  DOWNLOAD_NO_EXTRACT TRUE
  TLS_VERIFY TRUE
)
FetchContent_MakeAvailable(_gettext)
file(ARCHIVE_EXTRACT INPUT ${CMAKE_CURRENT_BINARY_DIR}/gettext-0.21.1.tar.gz)

ExternalProject_Add(gettext
  SOURCE_DIR ${CMAKE_CURRENT_BINARY_DIR}/gettext-0.21.1
  INSTALL_DIR ${PREFIX}
  CONFIGURE_COMMAND autoreconf -vfi && ./configure "--host=arm64-apple-darwin" "--prefix=" "--disable-shared" "--enable-static" "--disable-libasprintf" "--disable-java" "--disable-c++" CFLAGS=${DEFAULT_FLAGS} LDFLAGS=${DEFAULT_FLAGS}
  BUILD_COMMAND ${CMAKE_COMMAND} -E env
  make -C gettext-runtime/intl
  BUILD_IN_SOURCE TRUE
  INSTALL_COMMAND make -C gettext-runtime/intl DESTDIR=<INSTALL_DIR> install
)

#glib2.0
find_program (MESON meson REQUIRED)

FetchContent_Declare(_glib
  URL https://download.gnome.org/sources/glib/2.66/glib-2.66.8.tar.xz
  URL_HASH MD5=705dd46a43d339e8aea19e946e71c32a
  DOWNLOAD_DIR ${CMAKE_CURRENT_BINARY_DIR}
  DOWNLOAD_NO_EXTRACT TRUE
  TLS_VERIFY TRUE
)
FetchContent_MakeAvailable(_glib)
file(ARCHIVE_EXTRACT INPUT ${CMAKE_CURRENT_BINARY_DIR}/glib-2.66.8.tar.xz)

set(ARCHS "arm64")

foreach(ARCH IN LISTS ARCHS)
  ExternalProject_Add(glib${ARCH}
    DEPENDS libffi gettext
    SOURCE_DIR ${CMAKE_CURRENT_BINARY_DIR}/glib-2.66.8
    INSTALL_DIR ${PREFIX}
    PATCH_COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_CURRENT_SOURCE_DIR}/patches/meson-Fix-detection-of-static-libintl-on-macOS.patch ${CMAKE_CURRENT_SOURCE_DIR}/patches/fix-linkage-on-intl.patch ${CMAKE_CURRENT_SOURCE_DIR}/patches/only-keep-glib-and-gthread.patch <SOURCE_DIR> && patch -Np1 < <SOURCE_DIR>/meson-Fix-detection-of-static-libintl-on-macOS.patch || true && patch -Np1 < <SOURCE_DIR>/fix-linkage-on-intl.patch || true && patch -Np1 < <SOURCE_DIR>/only-keep-glib-and-gthread.patch || true
    CONFIGURE_COMMAND cmake -E env PKG_CONFIG_PATH=${PKG_CONFIG_PATH} CFLAGS=${DEFAULT_${ARCH}_GLIB_FLAGS} LDFLAGS=${DEFAULT_${ARCH}_GLIB_FLAGS} LIBRARY_PATH=${PREFIX}/lib
    ${MESON} setup --cross-file ${CMAKE_CURRENT_SOURCE_DIR}/ios.ini --prefix=${PREFIX}/${ARCH} --buildtype release --default-library static -Dinternal_pcre=true _build${ARCH}
    BUILD_COMMAND ${CMAKE_COMMAND} -E env CFLAGS=${DEFAULT_${ARCH}_GLIB_FLAGS} LDFLAGS=${DEFAULT_${ARCH}_GLIB_FLAGS}
    ninja -C _build${ARCH}
    BUILD_IN_SOURCE TRUE
    INSTALL_COMMAND ninja -C _build${ARCH} install all
  )
endforeach()

# Create a fat library. This will assume that libs/lib folder exist so it has to be crated
add_custom_target(glib
  DEPENDS glibarm64
#  COMMAND ${LIPO} ${PREFIX}/arm64/lib/libglib-2.0.a ${PREFIX}/armv7/lib/libglib-2.0.a ${PREFIX}/armv7s/lib/libglib-2.0.a -create -output ${PREFIX}/lib/libglib-2.0.a
  COMMAND cp -r ${PREFIX}/arm64/include/glib-2.0 ${PREFIX}/include && cp -r ${PREFIX}/arm64/lib/pkgconfig/* ${PKG_CONFIG_PATH}
)

#fluidsynth
FetchContent_Declare(_fluidsynth
  URL https://github.com/FluidSynth/fluidsynth/archive/refs/tags/v2.3.1.tar.gz
  URL_HASH MD5=7025da39fde375854bfb19ed38b7ee79
  DOWNLOAD_DIR ${CMAKE_CURRENT_BINARY_DIR}
  DOWNLOAD_NO_EXTRACT TRUE
  TLS_VERIFY TRUE
)
FetchContent_MakeAvailable(_fluidsynth)
file(ARCHIVE_EXTRACT INPUT ${CMAKE_CURRENT_BINARY_DIR}/v2.3.1.tar.gz)

ExternalProject_Add(fluidsynth
  SOURCE_DIR ${CMAKE_CURRENT_BINARY_DIR}/fluidsynth-2.3.1
  INSTALL_DIR ${PREFIX}
  PATCH_COMMAND ${CMAKE_COMMAND} -E copy_if_different ${CMAKE_CURRENT_SOURCE_DIR}/patches/fix_fluidsynth_link_against_static_glib.patch <SOURCE_DIR> && patch -p1 < <SOURCE_DIR>/fix_fluidsynth_link_against_static_glib.patch
  CONFIGURE_COMMAND ${CMAKE_COMMAND} -E env PKG_CONFIG_PATH=${PKG_CONFIG_PATH} CFLAGS=${DEFAULT_FLAGS} CXXFLAGS=${DEFAULT_FLAGS} LDFLAGS=${DEFAULT_FLAGS} LIBRARY_PATH=${PREFIX}/lib ${CMAKE_COMMAND} -S <SOURCE_DIR> -Denable-aufile=off -Denable-dbus=off -Denable-network=off -Denable-jack=off  -Denable-libsndfile=off -Denable-midishare=off -Denable-opensles=off -Denable-oboe=off -Denable-oss=off -Denable-dsound=off -Denable-waveout=off -Denable-winmidi=off -Denable-sdl2=off -Denable-pulseaudio=off -Denable-readline=off -Denable-lash=off -Denable-alsa=off -Denable-systemd=off -Denable-coreaudio=off -Denable-coremidi=off -Denable-framework=off -DBUILD_SHARED_LIBS=off -DLIB_SUFFIX= -DCMAKE_TOOLCHAIN_FILE=${CMAKE_CURRENT_SOURCE_DIR}/ios.toolchain.cmake
  BUILD_COMMAND ${CMAKE_COMMAND} -E env
  make
  INSTALL_COMMAND make DESTDIR=<INSTALL_DIR> install
)

#ExternalProject_Add(fluidsynth
#  SOURCE_DIR ${CMAKE_CURRENT_BINARY_DIR}/fluidsynth-2.3.1
#  INSTALL_DIR ${PREFIX}
#  CMAKE_ARGS
#  -Denable-aufile=off -Denable-dbus=off
#  -Denable-network=off -Denable-jack=off
#  -Denable-ladspa=off -Denable-libinstpatch=off
#  -Denable-libsndfile=off -Denable-midishare=off
#  -Denable-opensles=off -Denable-oboe=off
#  -Denable-oss=off -Denable-dsound=off
#  -Denable-waveout=off -Denable-winmidi=off
#  -Denable-sdl2=off -Denable-pulseaudio=off
#  -Denable-readline=off -Denable-lash=off
#  -Denable-alsa=off -Denable-systemd=off
#  -Denable-coreaudio=off -Denable-coremidi=off
#  -Denable-framework=off -Denable-dart=off
#  -DBUILD_SHARED_LIBS=off
#  -DLIB_SUFFIX=
#  -DCMAKE_TOOLCHAIN_FILE=${CMAKE_CURRENT_SOURCE_DIR}/ios.toolchain.cmake
#  -DCMAKE_C_FLAGS=${DEFAULT_FLAGS}
#  -DCMAKE_CXX_FLAGS=${DEFAULT_FLAGS}
#  -DCMAKE_EXE_LINKER_FLAGS=${DEFAULT_FLAGS}
#  -DCMAKE_INSTALL_PREFIX=${PREFIX}
#  -DCMAKE_INSTALL_LIBDIR=""
#  INSTALL_COMMAND make DESTDIR=<INSTALL_DIR> install
#)
