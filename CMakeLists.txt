cmake_minimum_required(VERSION 3.20)

# Do not allow building in root
if (CMAKE_BINARY_DIR STREQUAL CMAKE_SOURCE_DIR)
    message(FATAL_ERROR "Do not build in the root. Create a bin directory and remove ${CMAKE_SOURCE_DIR}/CMakeCache.txt")
endif ()

project(keeperfx C CXX)

set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 20)

include(FetchContent)
FetchContent_Declare(
  astronomy
  GIT_REPOSITORY https://github.com/cosinekitty/astronomy.git
  GIT_TAG master
  GIT_SHALLOW TRUE
)
FetchContent_MakeAvailable(astronomy)
add_library(astronomy_engine STATIC
  "${astronomy_SOURCE_DIR}/source/c/astronomy.c")
target_include_directories(astronomy_engine PUBLIC
  "${astronomy_SOURCE_DIR}/source/c")

FetchContent_Declare(
  centijson
  GIT_REPOSITORY https://github.com/mity/centijson.git
  GIT_TAG master
  GIT_SHALLOW TRUE
)
FetchContent_MakeAvailable(centijson)
add_library(centijson_static STATIC
  "${centijson_SOURCE_DIR}/src/json.c"
  "${centijson_SOURCE_DIR}/src/json-dom.c"
  "${centijson_SOURCE_DIR}/src/json-ptr.c"
  "${centijson_SOURCE_DIR}/src/value.c")
target_include_directories(centijson_static PUBLIC
  "${centijson_SOURCE_DIR}/src")

FetchContent_Declare(
  centitoml
  GIT_REPOSITORY https://github.com/SimLV/centitoml.git
  GIT_TAG main
  GIT_SHALLOW TRUE
)
FetchContent_MakeAvailable(centitoml)
set(CENTITOML_SOURCE_FILE "${CMAKE_SOURCE_DIR}/deps/centitoml/toml_api.c")
set(CENTITOML_INCLUDE_DIR "${CMAKE_SOURCE_DIR}/deps/centitoml")
if(EXISTS "${centitoml_SOURCE_DIR}/toml_api.c" AND EXISTS "${centitoml_SOURCE_DIR}/toml.h")
  set(CENTITOML_SOURCE_FILE "${centitoml_SOURCE_DIR}/toml_api.c")
  set(CENTITOML_INCLUDE_DIR "${centitoml_SOURCE_DIR}")
else()
  message(STATUS "Fetched centitoml has no parser sources; using local adapter in deps/centitoml")
endif()

# Get the abbreviated commit Id of the head.
find_package(Git REQUIRED)
execute_process(COMMAND "${GIT_EXECUTABLE}" describe --always OUTPUT_VARIABLE COMMIT_ID OUTPUT_STRIP_TRAILING_WHITESPACE)

set(VER_MAJOR       1)
set(VER_MINOR       2)
set(VER_RELEASE     0)
set(VER_BUILD       0)
set(VER_STRING      "${VER_MAJOR}.${VER_MINOR}.${VER_RELEASE}.${VER_BUILD} ${PACKAGE_SUFFIX}")
set(PACKAGE_SUFFIX  "")
set(GIT_REVISION    "${COMMIT_ID}")

# CMAKE_BINARY_DIR is defined in CMakePresets.json.
set(KEEPERFX_VER_DEFS_H_IN ${CMAKE_SOURCE_DIR}/ver_defs.h.in)
set(KEEPERFX_VER_DEFS_H_OUT ${CMAKE_BINARY_DIR}/ver_defs.h)
configure_file(${KEEPERFX_VER_DEFS_H_IN} ${KEEPERFX_VER_DEFS_H_OUT})
include_directories(
  ${CMAKE_BINARY_DIR}
  /usr/include/luajit-2.1/)

find_package(SDL2 CONFIG REQUIRED)
find_package(SDL2_image CONFIG REQUIRED)
find_package(SDL2_mixer CONFIG REQUIRED)
find_package(SDL2_net CONFIG REQUIRED)
find_package(ZLIB REQUIRED)
find_library(SPNG_LIBRARY NAMES spng REQUIRED)
find_library(MINIZIP_LIBRARY NAMES minizip REQUIRED)
find_library(LUAJIT_LIBRARY NAMES luajit-5.1 REQUIRED)
find_library(ENET_LIBRARY NAMES enet REQUIRED)
find_library(OPENAL_LIBRARY NAMES openal REQUIRED)
find_library(NATPMP_LIBRARY NAMES natpmp REQUIRED)
find_library(MINIUPNPC_LIBRARY NAMES miniupnpc REQUIRED)
find_library(AVCODEC_LIBRARY NAMES avcodec REQUIRED)
find_library(AVFORMAT_LIBRARY NAMES avformat REQUIRED)
find_library(AVUTIL_LIBRARY NAMES avutil REQUIRED)
find_library(SWRESAMPLE_LIBRARY NAMES swresample REQUIRED)

# Global definitions.
add_compile_definitions(_CRT_NONSTDC_NO_WARNINGS _CRT_SECURE_NO_WARNINGS)

file(GLOB_RECURSE KEEPERFX_SOURCES_C "src/*.c")
file(GLOB_RECURSE KEEPERFX_SOURCES_CXX "src/*.cpp")

list(REMOVE_ITEM KEEPERFX_SOURCES_CXX 
    "${CMAKE_CURRENT_SOURCE_DIR}/src/cdrom.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/steam_api.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/windows.cpp"
)

# Global definitions for all targets.
add_compile_definitions("DEBUG=$<IF:$<CONFIG:Debug>,1,0>")
add_compile_definitions("SPNG_STATIC=1")

# Add two executable targets: keeperfx and keeperfx_hvlog.
add_executable(keeperfx ${KEEPERFX_SOURCES_C} ${KEEPERFX_SOURCES_CXX})
target_sources(keeperfx PRIVATE "${CENTITOML_SOURCE_FILE}")
target_include_directories(keeperfx PRIVATE "${CENTITOML_INCLUDE_DIR}" "${centijson_SOURCE_DIR}/src")

target_compile_definitions(keeperfx PUBLIC BFDEBUG_LEVEL=0)
target_sources(keeperfx PRIVATE "res/keeperfx_stdres.rc")

add_executable(keeperfx_hvlog ${KEEPERFX_SOURCES_C} ${KEEPERFX_SOURCES_CXX})
target_sources(keeperfx_hvlog PRIVATE "${CENTITOML_SOURCE_FILE}")
target_include_directories(keeperfx_hvlog PRIVATE "${CENTITOML_INCLUDE_DIR}" "${centijson_SOURCE_DIR}/src")

target_compile_definitions(keeperfx_hvlog PUBLIC BFDEBUG_LEVEL=10)
target_sources(keeperfx_hvlog PRIVATE "res/keeperfx_stdres.rc")

message(STATUS "We are using ${CMAKE_CXX_COMPILER_ID}")

# The default bfd linker in MinGW is extremely slow. LLVM linker (LLD) is much much faster.
#set_property(TARGET keeperfx PROPERTY LINKER_TYPE LLD)
#set_property(TARGET keeperfx_hvlog PROPERTY LINKER_TYPE LLD)

set(WARNFLAGS -Wall -W -Wshadow -Wno-sign-compare -Wno-unused-parameter -Wno-strict-aliasing -Wno-unknown-pragmas -Werror -Wno-error=absolute-value -Wno-error=format-truncation= -O3)
set(GNU_COMPILER_FLAG -march=x86-64 -fno-omit-frame-pointer -fmessage-length=0)
#set(GNU_LINK_FLAG -Wl,--enable-auto-import)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wimplicit")
target_compile_options(keeperfx PRIVATE ${WARNFLAGS} ${GNU_COMPILER_FLAG})
target_compile_options(keeperfx_hvlog PRIVATE ${WARNFLAGS} ${GNU_COMPILER_FLAG})
target_link_options(keeperfx PRIVATE ${GNU_LINK_FLAG} -Wl,-Map,keeperfx.map)
target_link_options(keeperfx_hvlog PRIVATE ${GNU_LINK_FLAG} -Wl,-Map,keeperfx_hvlog.map)
#arget_link_libraries (keeperfx PUBLIC -static stdc++  -dynamic)
#target_link_libraries (keeperfx_hvlog PUBLIC -static stdc++  -dynamic)

# System libraries.
#target_link_libraries(keeperfx PRIVATE imagehlp dbghelp)
#target_link_libraries(keeperfx_hvlog PRIVATE imagehlp dbghelp)

# Go into submodules.
#add_subdirectory(deps)
#add_subdirectory(tools)

# External libraries.
target_link_libraries(keeperfx
    PRIVATE
    $<IF:$<TARGET_EXISTS:SDL2::SDL2>,SDL2::SDL2,SDL2::SDL2-static>)
target_link_libraries(keeperfx
    PRIVATE $<IF:$<TARGET_EXISTS:SDL2_mixer::SDL2_mixer>,SDL2_mixer::SDL2_mixer,SDL2_mixer::SDL2_mixer-static>)
target_link_libraries(keeperfx
    PRIVATE $<IF:$<TARGET_EXISTS:SDL2_net::SDL2_net>,SDL2_net::SDL2_net,SDL2_net::SDL2_net-static>)
target_link_libraries(keeperfx
    PRIVATE $<IF:$<TARGET_EXISTS:SDL2_image::SDL2_image>,SDL2_image::SDL2_image,SDL2_image::SDL2_image-static>)
target_link_libraries(keeperfx PRIVATE centijson_static)
target_link_libraries(keeperfx PRIVATE astronomy_engine)
target_link_libraries(keeperfx PRIVATE ZLIB::ZLIB)
target_link_libraries(keeperfx PRIVATE ${SPNG_LIBRARY} ${MINIZIP_LIBRARY} ${LUAJIT_LIBRARY})
target_link_libraries(keeperfx PRIVATE
  ${ENET_LIBRARY}
  ${OPENAL_LIBRARY}
  ${NATPMP_LIBRARY}
  ${MINIUPNPC_LIBRARY}
  ${AVCODEC_LIBRARY}
  ${AVFORMAT_LIBRARY}
  ${AVUTIL_LIBRARY}
  ${SWRESAMPLE_LIBRARY})

target_link_libraries(keeperfx_hvlog
    PRIVATE
    $<IF:$<TARGET_EXISTS:SDL2::SDL2>,SDL2::SDL2,SDL2::SDL2-static>)
target_link_libraries(keeperfx_hvlog
    PRIVATE $<IF:$<TARGET_EXISTS:SDL2_mixer::SDL2_mixer>,SDL2_mixer::SDL2_mixer,SDL2_mixer::SDL2_mixer-static>)
target_link_libraries(keeperfx_hvlog
    PRIVATE $<IF:$<TARGET_EXISTS:SDL2_net::SDL2_net>,SDL2_net::SDL2_net,SDL2_net::SDL2_net-static>)
target_link_libraries(keeperfx_hvlog
    PRIVATE $<IF:$<TARGET_EXISTS:SDL2_image::SDL2_image>,SDL2_image::SDL2_image,SDL2_image::SDL2_image-static>)
target_link_libraries(keeperfx_hvlog PRIVATE centijson_static)
target_link_libraries(keeperfx_hvlog PRIVATE astronomy_engine)
target_link_libraries(keeperfx_hvlog PRIVATE ZLIB::ZLIB)
target_link_libraries(keeperfx_hvlog PRIVATE ${SPNG_LIBRARY} ${MINIZIP_LIBRARY} ${LUAJIT_LIBRARY})
target_link_libraries(keeperfx_hvlog PRIVATE
  ${ENET_LIBRARY}
  ${OPENAL_LIBRARY}
  ${NATPMP_LIBRARY}
  ${MINIUPNPC_LIBRARY}
  ${AVCODEC_LIBRARY}
  ${AVFORMAT_LIBRARY}
  ${AVUTIL_LIBRARY}
  ${SWRESAMPLE_LIBRARY})
