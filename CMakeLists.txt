cmake_minimum_required(VERSION 3.22.1)

project("retro")
execute_process(
        COMMAND git rev-parse --short HEAD
        OUTPUT_VARIABLE GIT_VERSION
        ERROR_QUIET
        OUTPUT_STRIP_TRAILING_WHITESPACE
)

if(GIT_VERSION)
    add_compile_definitions(GIT_VERSION="${GIT_VERSION}")
endif()

add_compile_definitions(
        __LIBRETRO__
        PATH_MAX=1024
        FCEU_VERSION_NUMERIC=9813
        FRONTEND_SUPPORTS_RGB565
        HAVE_NTSC_FILTER
)

set(CORE_DIR src)
set(LIBRETRO_COMM_DIR ${CORE_DIR}/drivers/libretro/libretro-common)

set(FCEU_SRC_DIRS
        "${CORE_DIR}/boards"
        "${CORE_DIR}/input"
)

set(SOURCES_C "")
foreach(dir ${FCEU_SRC_DIRS})
    file(GLOB temp_sources "${dir}/*.c")
    list(APPEND SOURCES_C ${temp_sources})
endforeach()

list(APPEND SOURCES_C
        ${CORE_DIR}/drivers/libretro/libretro.c
        ${CORE_DIR}/drivers/libretro/libretro_dipswitch.c
        ${CORE_DIR}/cart.c
        ${CORE_DIR}/cheat.c
        ${CORE_DIR}/crc32.c
        ${CORE_DIR}/fceu-endian.c
        ${CORE_DIR}/fceu-memory.c
        ${CORE_DIR}/fceu.c
        ${CORE_DIR}/fds.c
        ${CORE_DIR}/fds_apu.c
        ${CORE_DIR}/file.c
        ${CORE_DIR}/filter.c
        ${CORE_DIR}/general.c
        ${CORE_DIR}/input.c
        ${CORE_DIR}/md5.c
        ${CORE_DIR}/nsf.c
        ${CORE_DIR}/palette.c
        ${CORE_DIR}/ppu.c
        ${CORE_DIR}/sound.c
        ${CORE_DIR}/state.c
        ${CORE_DIR}/video.c
        ${CORE_DIR}/vsuni.c
        ${CORE_DIR}/ines.c
        ${CORE_DIR}/unif.c
        ${CORE_DIR}/x6502.c
        ${CORE_DIR}/ntsc/nes_ntsc.c
)

if (NOT STATIC_LINKING STREQUAL "1")
    list(APPEND SOURCES_C
            ${LIBRETRO_COMM_DIR}/compat/compat_posix_string.c
            ${LIBRETRO_COMM_DIR}/compat/compat_snprintf.c
            ${LIBRETRO_COMM_DIR}/compat/compat_strcasestr.c
            ${LIBRETRO_COMM_DIR}/compat/compat_strl.c
            ${LIBRETRO_COMM_DIR}/compat/fopen_utf8.c
            ${LIBRETRO_COMM_DIR}/encodings/encoding_utf.c
            ${LIBRETRO_COMM_DIR}/file/file_path.c
            ${LIBRETRO_COMM_DIR}/file/file_path_io.c
            ${LIBRETRO_COMM_DIR}/streams/file_stream.c
            ${LIBRETRO_COMM_DIR}/streams/file_stream_transforms.c
            ${LIBRETRO_COMM_DIR}/streams/memory_stream.c
            ${LIBRETRO_COMM_DIR}/string/stdstring.c
            ${LIBRETRO_COMM_DIR}/time/rtime.c
            ${LIBRETRO_COMM_DIR}/vfs/vfs_implementation.c
    )
endif ()

set(INCLUDES
        ${CORE_DIR}/drivers/libretro
        ${LIBRETRO_COMM_DIR}/include
        ${CORE_DIR}
        ${CORE_DIR}/input
        ${CORE_DIR}/boards
        ${CORE_DIR}/ntsc
)

if (${platform} MATCHES "msvc2003")
    list(APPEND INCLUDES ${LIBRETRO_COMM_DIR}/include/compat/msvc)
endif ()
include_directories(${INCLUDES})

add_library(retro SHARED ${SOURCES_C})