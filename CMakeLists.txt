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
set(DEFAULT_FLAGS "${CMAKE_C_FLAGS} -isysroot ${CMAKE_OSX_SYSROOT_INT} -miphoneos-version-min=7.0 -arch arm64 -arch armv7 -arch armv7s")

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
