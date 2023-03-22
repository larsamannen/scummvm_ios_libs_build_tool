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

find_program(XCODEBUILD_EXECUTABLE xcodebuild)
if(NOT XCODEBUILD_EXECUTABLE)
  message(FATAL_ERROR "xcodebuild not found. Please install either the standalone commandline tools or Xcode.")
endif()

set(SDK_NAME iphoneos)
set(TOOLCHAIN)

execute_process(COMMAND ${XCODEBUILD_EXECUTABLE} -version -sdk ${SDK_NAME} Path
  OUTPUT_VARIABLE CMAKE_OSX_SYSROOT_INT
  ERROR_QUIET
  OUTPUT_STRIP_TRAILING_WHITESPACE)
set(CMAKE_OSX_SYSROOT ${CMAKE_OSX_SYSROOT_INT})

set(PREFIX ${CMAKE_CURRENT_SOURCE_DIR}/libs)
set(DEFAULT_FLAGS "${CMAKE_C_FLAGS} -isysroot ${CMAKE_OSX_SYSROOT} -miphoneos-version-min=7.0 -arch arm64 -arch armv7 -arch armv7s")

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
  CC=${CMAKE_C_COMPILER}
  make libbz2.a CFLAGS=${DEFAULT_FLAGS} "-Wall -Winline -O2 -D_FILE_OFFSET_BITS=64" LDFLAGS=${DEFAULT_FLAGS}
  BUILD_IN_SOURCE TRUE
  INSTALL_COMMAND mkdir <INSTALL_DIR>/lib && cp -f libbz2.a <INSTALL_DIR>/lib && mkdir <INSTALL_DIR>/include && cp -f bzlib.h <INSTALL_DIR>/include
)
