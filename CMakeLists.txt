cmake_minimum_required(VERSION 3.20)

# Do not allow building in root
if (CMAKE_BINARY_DIR STREQUAL CMAKE_SOURCE_DIR)
    message(FATAL_ERROR "Do not build in the root. Create a bin directory and remove ${CMAKE_SOURCE_DIR}/CMakeCache.txt")
endif ()

project(keeperfx 
	LANGUAGES C CXX
	VERSION 1.3.1.0
	HOMEPAGE_URL "https://keeperfx.net/"
)

set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 20)



# Get the abbreviated commit Id of the head.
find_package(Git REQUIRED)
execute_process(COMMAND "${GIT_EXECUTABLE}" describe --always OUTPUT_VARIABLE COMMIT_ID OUTPUT_STRIP_TRAILING_WHITESPACE)

set(VER_MAJOR       ${PROJECT_VERSION_MAJOR})
set(VER_MINOR       ${PROJECT_VERSION_MINOR})
set(VER_RELEASE     ${PROJECT_VERSION_PATCH})
set(VER_BUILD       ${PROJECT_VERSION_TWEAK})
set(VER_STRING      "${VER_MAJOR}.${VER_MINOR}.${VER_RELEASE}.${VER_BUILD} ${PACKAGE_SUFFIX}")
set(PACKAGE_SUFFIX  "")
set(GIT_REVISION    "${COMMIT_ID}")

# CMAKE_BINARY_DIR is defined in CMakePresets.json.
set(KEEPERFX_VER_DEFS_H_IN ${CMAKE_SOURCE_DIR}/include/ver_defs.h.in)
set(KEEPERFX_VER_DEFS_H_OUT ${CMAKE_BINARY_DIR}/include/ver_defs.h)
configure_file(${KEEPERFX_VER_DEFS_H_IN} ${KEEPERFX_VER_DEFS_H_OUT})
include_directories(
  ${CMAKE_BINARY_DIR}
)



# Global definitions.
add_compile_definitions(_CRT_NONSTDC_NO_WARNINGS _CRT_SECURE_NO_WARNINGS)


# Global definitions for all targets.
add_compile_definitions("DEBUG=$<IF:$<CONFIG:Debug>,1,0>")
add_compile_definitions("SPNG_STATIC=1")

# Go into submodules.
#add_subdirectory(deps)
add_subdirectory(deps_linux)
add_subdirectory(src)
#add_subdirectory(tools)

# Package
set(CPACK_DEBIAN_PACKAGE_MAINTAINER "https://github.com/dkfans/keeperfx")
set(CPACK_DEBIAN_FILE_NAME DEB-DEFAULT) 
include(CPack)
