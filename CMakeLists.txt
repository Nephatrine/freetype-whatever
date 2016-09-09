cmake_minimum_required(VERSION 3.3.2)

project(freetype2_we
	VERSION 2.7.0)

set(FREETYPE_SOVERSION ${PROJECT_VERSION_MINOR})

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/config/CMake")
set(WHATEVER_SET_LANGUAGE C)
include(WhatEver)

we_mark_license(FTL.TXT)
we_mark_docs(README.md)

#
# Project Source Code
#

list(APPEND FREETYPE_SOURCE
	"${CMAKE_CURRENT_SOURCE_DIR}/src/base/ftbase.c"
	"${CMAKE_CURRENT_SOURCE_DIR}/src/base/ftbbox.c"
	"${CMAKE_CURRENT_SOURCE_DIR}/src/base/ftbitmap.c"
	"${CMAKE_CURRENT_SOURCE_DIR}/src/base/ftcid.c"
	"${CMAKE_CURRENT_SOURCE_DIR}/src/base/ftdebug.c"
	"${CMAKE_CURRENT_SOURCE_DIR}/src/base/ftfntfmt.c"
	"${CMAKE_CURRENT_SOURCE_DIR}/src/base/ftfstype.c"
	"${CMAKE_CURRENT_SOURCE_DIR}/src/base/ftgasp.c"
	"${CMAKE_CURRENT_SOURCE_DIR}/src/base/ftglyph.c"
	"${CMAKE_CURRENT_SOURCE_DIR}/src/base/ftgxval.c"
	"${CMAKE_CURRENT_SOURCE_DIR}/src/base/ftinit.c"
	"${CMAKE_CURRENT_SOURCE_DIR}/src/base/ftlcdfil.c"
	"${CMAKE_CURRENT_SOURCE_DIR}/src/base/ftotval.c"
	"${CMAKE_CURRENT_SOURCE_DIR}/src/base/ftpatent.c"
	"${CMAKE_CURRENT_SOURCE_DIR}/src/base/ftstroke.c"
	"${CMAKE_CURRENT_SOURCE_DIR}/src/base/ftsynth.c"
	"${CMAKE_CURRENT_SOURCE_DIR}/src/base/ftsystem.c"
	"${CMAKE_CURRENT_SOURCE_DIR}/src/base/fttype1.c"
	"${CMAKE_CURRENT_SOURCE_DIR}/src/autofit/autofit.c"
	"${CMAKE_CURRENT_SOURCE_DIR}/src/cache/ftcache.c"
	"${CMAKE_CURRENT_SOURCE_DIR}/src/cff/cff.c"
	"${CMAKE_CURRENT_SOURCE_DIR}/src/gxvalid/gxvalid.c"
	"${CMAKE_CURRENT_SOURCE_DIR}/src/otvalid/otvalid.c"
	"${CMAKE_CURRENT_SOURCE_DIR}/src/pshinter/pshinter.c"
	"${CMAKE_CURRENT_SOURCE_DIR}/src/psnames/psnames.c"
	"${CMAKE_CURRENT_SOURCE_DIR}/src/raster/raster.c"
	"${CMAKE_CURRENT_SOURCE_DIR}/src/sfnt/sfnt.c"
	"${CMAKE_CURRENT_SOURCE_DIR}/src/smooth/smooth.c"
	"${CMAKE_CURRENT_SOURCE_DIR}/src/truetype/truetype.c")

#
# External Modules
#

find_package(BZip2)
find_package(PNG)
find_package(ZLIB)

#
# Project Options
#

option(FT_CONFIG_OPTION_DISABLE_STREAM_SUPPORT "Do Not Support Stream Functions & Types" OFF)
option(FT_CONFIG_OPTION_ENVIRONMENT_PROPERTIES "Support FREETYPE_PROPERTIES Environment Variable" OFF)
option(FT_CONFIG_OPTION_INCREMENTAL "Support FT_Incremental_Interface" OFF)
option(FT_CONFIG_OPTION_MAC_FONTS "Support Macintosh Outline Formats" OFF)

cmake_dependent_option(FT_CONFIG_OPTION_SYSTEM_ZLIB "Use System ZLib Library" ON ZLIB_FOUND OFF)
cmake_dependent_option(FT_CONFIG_OPTION_USE_PNG "Support PNG Bitmap Font Data" OFF PNG_FOUND OFF)

option(FT_DRIVER_BDF "Support Adobe Glyph Bitmap Fonts" OFF)
if(FT_DRIVER_BDF)
	list(APPEND FREETYPE_SOURCE
		"${CMAKE_CURRENT_SOURCE_DIR}/src/base/ftbdf.c"
		"${CMAKE_CURRENT_SOURCE_DIR}/src/bdf/bdf.c")
endif()

option(FT_DRIVER_PCF "Support CyberTex PCF Bitmap Fonts" OFF)
if(FT_DRIVER_PCF)
	list(APPEND FREETYPE_SOURCE
		"${CMAKE_CURRENT_SOURCE_DIR}/src/pcf/pcf.c")
endif()

option(FT_DRIVER_TRUEDOC "Support BitStream TrueDoc Fonts (.PFR)" OFF)
if(FT_DRIVER_TRUEDOC)
	list(APPEND FREETYPE_SOURCE
		"${CMAKE_CURRENT_SOURCE_DIR}/src/base/ftpfr.c"
		"${CMAKE_CURRENT_SOURCE_DIR}/src/pfr/pfr.c")
endif()

option(FT_DRIVER_TYPE1 "Support PostScript Type 1 Fonts" OFF)
if(FT_DRIVER_TYPE1)
	list(APPEND FREETYPE_SOURCE
		"${CMAKE_CURRENT_SOURCE_DIR}/src/base/ftmm.c"
		"${CMAKE_CURRENT_SOURCE_DIR}/src/cid/type1cid.c"
		"${CMAKE_CURRENT_SOURCE_DIR}/src/psaux/psaux.c"
		"${CMAKE_CURRENT_SOURCE_DIR}/src/type1/type1.c"
		"${CMAKE_CURRENT_SOURCE_DIR}/src/type42/type42.c")
endif()

option(FT_DRIVER_WINFNT "Support Windows Bitmap Fonts (.FON, .FNT)" OFF)
if(FT_DRIVER_WINFNT)
	list(APPEND FREETYPE_SOURCE
		"${CMAKE_CURRENT_SOURCE_DIR}/src/base/ftwinfnt.c"
		"${CMAKE_CURRENT_SOURCE_DIR}/src/winfonts/winfnt.c")
endif()

cmake_dependent_option(FT_CONFIG_OPTION_USE_BZIP2 "Support BZip2 (.bz2) Compression" OFF BZIP2_FOUND OFF)
if(FT_CONFIG_OPTION_USE_BZIP2)
	list(APPEND FREETYPE_SOURCE
		"${CMAKE_CURRENT_SOURCE_DIR}/src/bzip2/ftbzip2.c")
endif()

option(FT_CONFIG_OPTION_USE_LZW "Support LZW (.Z) Compression" OFF)
if(FT_CONFIG_OPTION_USE_LZW)
	list(APPEND FREETYPE_SOURCE
		"${CMAKE_CURRENT_SOURCE_DIR}/src/lzw/ftlzw.c")
endif()

option(FT_CONFIG_OPTION_USE_ZLIB "Support ZLib (.gz) Compression" OFF)
if(FT_CONFIG_OPTION_USE_ZLIB)
	list(APPEND FREETYPE_SOURCE
		"${CMAKE_CURRENT_SOURCE_DIR}/src/gzip/ftgzip.c")
endif()

#
# Compiler Tests
#

if(NOT MSVC)
	we_request_compile_flag("-march=native")
	we_request_compile_flag("-Wextra")
	we_request_link_flag("-Wl,--as-needed")
	we_request_link_flag("-Wl,--no-undefined")
	we_request_link_flag("-Wl,--no-allow-shlib-undefined")

	if(MINGW)
		we_request_link_flag("-Wl,--nxcompat")
		we_request_link_flag("-Wl,--dynamicbase")
		we_request_link_flag("-Wl,--high-entropy-va")
	endif()
endif()

we_check_include("fcntl.h")
we_check_include("stdint.h")
we_check_include("unistd.h")

#
# Generate Configuration Headers
#

if("${CMAKE_BUILD_TYPE}" STREQUAL "Debug")
	set(FT_DEBUG_LEVEL_ERROR ON)
	set(FT_DEBUG_LEVEL_TRACE ON)
	set(FT_DEBUG_AUTOFIT ON)
	set(FT_DEBUG_MEMORY ON)
endif()

we_generate_configs("${CMAKE_CURRENT_SOURCE_DIR}/config"
	"${CMAKE_CURRENT_BINARY_DIR}/include/freetype/config")
list(APPEND FREETYPE_HEADERS ${WHATEVER_GENERATED})

#
# Build Project Libraries
#

option(BUILD_LIBRARY_STATIC "Build Static Library" ON)
if(BUILD_LIBRARY_STATIC)
	we_build_library_static(FREETYPE_STATIC ${PROJECT_NAME} ${FREETYPE_SOVERSION} "${FREETYPE_SOURCE};${FREETYPE_HEADERS}")
endif()

option(BUILD_LIBRARY_SHARED "Build Shared Library" ON)
if(BUILD_LIBRARY_STATIC)
	we_build_library_shared(FREETYPE_SHARED ${PROJECT_NAME} ${FREETYPE_SOVERSION} "${FREETYPE_SOURCE};${FREETYPE_HEADERS}")
endif()

we_include_public("${WHATEVER_TARGETS_LIBRARY}" include "${PROJECT_NAME}-${FREETYPE_SOVERSION}")

#
# Link External Libraries
#

list(APPEND FREETYPE_EXT_DEF FT2_BUILD_LIBRARY)

if(FT_CONFIG_OPTION_USE_BZIP2)
	list(APPEND FREETYPE_EXT_LIB ${BZIP2_LIBRARIES})
	list(APPEND FREETYPE_EXT_INC ${BZIP2_INCLUDE_DIR})
endif()

if(FT_CONFIG_OPTION_USE_PNG)
	list(APPEND FREETYPE_EXT_LIB ${PNG_LIBRARIES})
	list(APPEND FREETYPE_EXT_INC ${PNG_INCLUDE_DIRS})
	list(APPEND FREETYPE_EXT_DEF ${PNG_DEFINITIONS})
endif()

if(FT_CONFIG_OPTION_USE_ZLIB AND FT_CONFIG_OPTION_SYSTEM_ZLIB)
	list(APPEND FREETYPE_EXT_LIB ${ZLIB_LIBRARIES})
	list(APPEND FREETYPE_EXT_INC ${ZLIB_INCLUDE_DIRS})
endif()

we_external_include("${WHATEVER_TARGETS_LIBRARY}" PRIVATE "${FREETYPE_EXT_INC}")
we_external_defines("${WHATEVER_TARGETS_LIBRARY}" PRIVATE "${FREETYPE_EXT_DEF}")
we_external_library("${WHATEVER_TARGETS_SHARED}" PRIVATE "${FREETYPE_EXT_LIB}")
we_external_library("${WHATEVER_TARGETS_STATIC}" PUBLIC "${FREETYPE_EXT_LIB}")

#
# Create Installation Packages
#

we_package(freetype Nephatrine nephatrine@gmail.com "Open Source Font Engine")

