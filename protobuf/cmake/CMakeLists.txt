cmake_minimum_required(VERSION 2.8)

project(protobuf C CXX)

option(BUILD_TESTING "Build tests" ON)
option(BUILD_SHARED_LIBS "Build Shared Libraries" OFF)
if (MSVC)
  option(ZLIB "Build with zlib support" OFF)
endif (MSVC)

add_definitions(-DGOOGLE_PROTOBUF_CMAKE_BUILD)

find_package(Threads REQUIRED)
if (CMAKE_USE_PTHREADS_INIT)
  add_definitions(-DHAVE_PTHREAD)
endif (CMAKE_USE_PTHREADS_INIT)

if (MSVC)
  if (ZLIB)
    set(HAVE_ZLIB 1)
    find_path(ZLIB_INCLUDE_DIRECTORIES zlib.h ${protobuf_SOURCE_DIR})
    find_library(ZLIB_LIBRARIES zdll ${protobuf_SOURCE_DIR})
  else (ZLIB)
    set(HAVE_ZLIB 0)
  endif (ZLIB)
else (MSVC)
  find_package(ZLIB)
  if (ZLIB_FOUND)
    set(HAVE_ZLIB 1)
  else (ZLIB_FOUND)
    set(HAVE_ZLIB 0)
    # Explicitly set these to empty (override NOT_FOUND) so cmake doesn't
    # complain when we use them later.
    set(ZLIB_INCLUDE_DIRECTORIES)
    set(ZLIB_LIBRARIES)
  endif (ZLIB_FOUND)
endif (MSVC)

if (HAVE_ZLIB)
  add_definitions(-DHAVE_ZLIB)
endif (HAVE_ZLIB)

if (MSVC)
  if (BUILD_SHARED_LIBS)
    add_definitions(-DPROTOBUF_USE_DLLS)
  endif (BUILD_SHARED_LIBS)
  add_definitions(/wd4244 /wd4267 /wd4018 /wd4355 /wd4800 /wd4251 /wd4996 /wd4146 /wd4305)
endif (MSVC)

if (MSVC)
  string(REPLACE "/" "\\" PROTOBUF_SOURCE_WIN32_PATH ${protobuf_SOURCE_DIR})
  string(REPLACE "/" "\\" PROTOBUF_BINARY_WIN32_PATH ${protobuf_BINARY_DIR})
  configure_file(extract_includes.bat.in extract_includes.bat)
endif (MSVC)

get_filename_component(protobuf_source_dir ${protobuf_SOURCE_DIR} PATH)

include_directories(
  ${ZLIB_INCLUDE_DIRECTORIES}
  ${protobuf_BINARY_DIR}
  ${protobuf_source_dir}/src)

if (MSVC)
  # Add the "lib" prefix for generated .lib outputs.
  set(LIB_PREFIX lib)
else (MSVC)
  # When building with "make", "lib" prefix will be added automatically by
  # the build tool.
  set(LIB_PREFIX)
endif (MSVC)

include(libprotobuf-lite.cmake)
include(libprotobuf.cmake)
include(libprotoc.cmake)
include(protoc.cmake)
if (BUILD_TESTING)
  include(tests.cmake)
endif (BUILD_TESTING)
