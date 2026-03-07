# astronomy (github)
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

# centijson (github)
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
target_include_directories(centijson_static INTERFACE
  "${centijson_SOURCE_DIR}/src")

# centitoml (local)
set(centitoml_SOURCE_DIR "${PROJECT_SOURCE_DIR}/deps/centitoml/")
add_library(centitoml_static STATIC
  	"${centitoml_SOURCE_DIR}/toml_api.c"
)
target_include_directories(centitoml_static INTERFACE "${centitoml_SOURCE_DIR}")
target_link_libraries(centitoml_static PRIVATE centijson_static)

# luajit (os)
find_package(PkgConfig QUIET REQUIRED)
pkg_check_modules(LUAJIT REQUIRED IMPORTED_TARGET GLOBAL luajit)
add_library(luajit ALIAS PkgConfig::LUAJIT)

# others
find_library(SPNG_LIBRARY NAMES spng REQUIRED)
find_library(MINIZIP_LIBRARY NAMES minizip REQUIRED)
find_library(ENET_LIBRARY NAMES enet REQUIRED)
find_library(OPENAL_LIBRARY NAMES openal REQUIRED)
find_library(NATPMP_LIBRARY NAMES natpmp REQUIRED)
find_library(MINIUPNPC_LIBRARY NAMES miniupnpc REQUIRED)
find_library(AVCODEC_LIBRARY NAMES avcodec REQUIRED)
find_library(AVFORMAT_LIBRARY NAMES avformat REQUIRED)
find_library(AVUTIL_LIBRARY NAMES avutil REQUIRED)
find_library(SWRESAMPLE_LIBRARY NAMES swresample REQUIRED)