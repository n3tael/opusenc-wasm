cmake_minimum_required(VERSION 3.23)
project(opusenc_wasm VERSION 1.0)

if(NOT CMAKE_INSTALL_PACKAGEDIR)
    set(CMAKE_INSTALL_PACKAGEDIR "${CMAKE_INSTALL_LIBDIR}/cmake/opus")
endif()

set(OPUS_BUILD_PROGRAMS OFF CACHE BOOL "" FORCE)
set(OPUS_BUILD_TESTING OFF CACHE BOOL "" FORCE)
set(BUILD_PROGRAMS OFF CACHE BOOL "" FORCE)
set(BUILD_TESTING OFF CACHE BOOL "" FORCE)
set(BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)
set(BUILD_DOCS OFF CACHE BOOL "" FORCE)
set(INSTALL_MANPAGES OFF CACHE BOOL "" FORCE)
set(BUILD_CXXLIBS OFF CACHE BOOL "" FORCE)

add_subdirectory(external/ogg)
add_subdirectory(external/flac)
add_subdirectory(external/opus)
# --- libopusenc ---
set(LIBOPUSENC_SOURCES
    external/libopusenc/src/ogg_packer.c
    external/libopusenc/src/opus_header.c
    external/libopusenc/src/opusenc.c
    external/libopusenc/src/picture.c
    external/libopusenc/src/resample.c
    external/libopusenc/src/unicode_support.c
)

add_library(libopusenc STATIC ${LIBOPUSENC_SOURCES})

target_compile_definitions(libopusenc PRIVATE
    OUTSIDE_SPEEX
    RANDOM_PREFIX=l2o
    PACKAGE_NAME="libopusenc"
    PACKAGE_VERSION="0.3"
)

target_include_directories(libopusenc PUBLIC 
    external/libopusenc/include
    external/opus/include
)

target_include_directories(libopusenc PRIVATE 
    external/libopusenc/src
)

target_link_libraries(libopusenc PRIVATE opus ogg)

# --- opusenc ---
set(OPUSENC_SOURCES
    external/opus-tools/src/opusenc.c
    external/opus-tools/src/audio-in.c
    external/opus-tools/src/diag_range.c
    external/opus-tools/src/wav_io.c
    external/opus-tools/src/opus_header.c
    external/opus-tools/src/flac.c
    external/opus-tools/src/tagcompare.c
)

find_package(Git)

if(GIT_EXECUTABLE)
  execute_process(
    COMMAND ${GIT_EXECUTABLE} --git-dir=${CMAKE_CURRENT_LIST_DIR}/external/opus-tools/.git describe --tags --match "v*" --dirty
    OUTPUT_VARIABLE OPUSENC_VERSION
    OUTPUT_STRIP_TRAILING_WHITESPACE
    )
endif()

if(OPUSENC_VERSION STREQUAL "")
  set(OPUSENC_VERSION 0.0.0-unknown)
  message(WARNING "Could not get package version.")
endif()

add_executable(opusenc ${OPUSENC_SOURCES})

target_compile_definitions(opusenc PRIVATE
    PACKAGE_NAME="opus-tools"
	PACKAGE_VERSION="${OPUSENC_VERSION}"
    HAVE_LIBFLAC
)

set_target_properties(opusenc PROPERTIES LINK_FLAGS 
    "-Os -s MODULARIZE=1 -s EXPORT_ES6=1 -s EXIT_RUNTIME=1 -s INVOKE_RUN=0 -s FORCE_FILESYSTEM=1 -s EXPORTED_RUNTIME_METHODS=['FS','callMain'] -s ALLOW_MEMORY_GROWTH=1 -s ENVIRONMENT='worker' -s PROXY_TO_PTHREAD=1 -s PTHREAD_POOL_SIZE=1 -pthread -s STACK_SIZE=16MB -s INITIAL_MEMORY=32MB")

target_link_libraries(opusenc PRIVATE FLAC opus libopusenc)