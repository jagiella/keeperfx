
# source files
file(GLOB_RECURSE KEEPERFX_SOURCES_C "*.c")
file(GLOB_RECURSE KEEPERFX_SOURCES_CXX "*.cpp")

list(REMOVE_ITEM KEEPERFX_SOURCES_CXX 
    "${CMAKE_CURRENT_SOURCE_DIR}/cdrom.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/steam_api.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/windows.cpp"
)

# Add two executable targets: keeperfx and keeperfx_hvlog.
add_executable(keeperfx ${KEEPERFX_SOURCES_C} ${KEEPERFX_SOURCES_CXX})
target_sources(keeperfx PRIVATE "${CENTITOML_SOURCE_FILE}")
target_include_directories(keeperfx PRIVATE "${CENTITOML_INCLUDE_DIR}" "${centijson_SOURCE_DIR}/src")

target_compile_definitions(keeperfx PUBLIC BFDEBUG_LEVEL=0)
#target_compile_definitions(keeperfx_hvlog PUBLIC BFDEBUG_LEVEL=10)
target_sources(keeperfx PRIVATE "../res/keeperfx_stdres.rc")




message(STATUS "We are using ${CMAKE_CXX_COMPILER_ID}")

# The default bfd linker in MinGW is extremely slow. LLVM linker (LLD) is much much faster.
set_property(TARGET keeperfx PROPERTY LINKER_TYPE LLD)

set(WARNFLAGS -Wall -W -Wshadow -Wno-sign-compare -Wno-unused-parameter -Wno-strict-aliasing -Wno-unknown-pragmas -Werror -Wno-error=absolute-value -Wno-error=format-truncation= -O3)
set(GNU_COMPILER_FLAG -march=x86-64 -fno-omit-frame-pointer -fmessage-length=0)
#set(GNU_LINK_FLAG -Wl,--enable-auto-import)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wimplicit")
target_compile_options(keeperfx PRIVATE ${WARNFLAGS} ${GNU_COMPILER_FLAG})
target_link_options(keeperfx PRIVATE ${GNU_LINK_FLAG} -Wl,-Map,keeperfx.map)
#arget_link_libraries (keeperfx PUBLIC -static stdc++  -dynamic)

# System libraries.
#target_link_libraries(keeperfx PRIVATE imagehlp dbghelp)

# External libraries.
find_package(SDL2 CONFIG REQUIRED)
find_package(SDL2_image CONFIG REQUIRED)
find_package(SDL2_mixer CONFIG REQUIRED)
find_package(SDL2_net CONFIG REQUIRED)
find_package(ZLIB REQUIRED)

target_link_libraries(keeperfx
    PRIVATE
    # local (build)
    centitoml_static
	astronomy_engine
	# os (find_package)
    SDL2::SDL2 
    SDL2_mixer::SDL2_mixer 
    SDL2_net::SDL2_net 
    SDL2_image::SDL2_image
	ZLIB::ZLIB
	# os (pkgconfig)
	luajit
	# os (find_library)
	${SPNG_LIBRARY} 
	${MINIZIP_LIBRARY} 
	${ENET_LIBRARY}
	${OPENAL_LIBRARY}
	${NATPMP_LIBRARY}
	${MINIUPNPC_LIBRARY}
	${AVCODEC_LIBRARY}
	${AVFORMAT_LIBRARY}
	${AVUTIL_LIBRARY}
	${SWRESAMPLE_LIBRARY}
)
