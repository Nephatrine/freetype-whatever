cmake_minimum_required(VERSION 3.4)
cmake_policy(SET CMP0048 NEW)
cmake_policy(SET CMP0054 NEW)
cmake_policy(SET CMP0063 NEW)
if(WIN32)
	set(CPACK_GENERATOR "ZIP")
else()
	set(CPACK_GENERATOR "TGZ;DEB")
endif()

#
# Project Definition
#

project(freetype2
	VERSION 2.6.5
	LANGUAGES C)
set(FT2_SOVERSION "2.6")
set(CPACK_PACKAGE_CONTACT "nephatrine@gmail.com")
set(CPACK_PACKAGE_DESCRIPTION_FILE "${CMAKE_CURRENT_SOURCE_DIR}/README.md")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Free, High-Quality, & Portable Font Engine")
set(CPACK_PACKAGE_INSTALL_DIRECTORY "libFreeType")
set(CPACK_PACKAGE_NAME "libfreetype")
set(CPACK_PACKAGE_VENDOR "Daniel Wolf")
set(CPACK_PACKAGE_VERSION_MAJOR "${PROJECT_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${PROJECT_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${PROJECT_VERSION_PATCH}")
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE.md")

#
# CMake Modules
#

include(CheckCCompilerFlag)
include(CheckIncludeFile)
include(CMakeDependentOption)
include(CPack)
include(GNUInstallDirs)

#
# External Requirements
#

list(APPEND CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/config/CMake/modules")
find_package(BZip2)
#find_package(HarfBuzz)
find_package(PNG)
find_package(ZLIB)

#
# Project Options
#

set(CMAKE_C_STANDARD "99")
set(CMAKE_C_VISIBILITY_PRESET hidden)
set(CMAKE_VISIBILITY_INLINES_HIDDEN 1)

option(FT2_BUILD_SHARED "Build Shared Library" ON)
option(FT2_BUILD_STATIC "Build Static Library" ON)
cmake_dependent_option(FT2_BUILD_TESTS "Build Tests" ON "NOT CMAKE_TOOLCHAIN_FILE" OFF)

option(FT2_DRIVER_BDF "Support Adobe Glyph Bitmap Fonts" OFF)
option(FT2_DRIVER_CCF "Support OpenType/CFF Fonts (.OTF)" OFF)
option(FT2_DRIVER_MACFNT "Support Macintosh Outline Formats" OFF)
option(FT2_DRIVER_PCF "Support CyberTex PCF Bitmap Fonts" OFF)
option(FT2_DRIVER_TRUEDOC "Support BitStream TrueDoc Fonts (.PFR)" OFF)
option(FT2_DRIVER_TRUETYPE "Support TrueType Fonts (.TTF)" ON)
option(FT2_DRIVER_TYPE1 "Support PostScript Type 1 Fonts" OFF)
option(FT2_DRIVER_WINFNT "Support Windows Bitmap Fonts (.FON, .FNT)" OFF)

option(FT2_SUPPORT_CACHE "Support FreeType Cache (Experimental)" ON)
option(FT2_SUPPORT_ENVPROP "Support Envinronment Properties" OFF)
option(FT2_SUPPORT_INCREMENTAL "Support Incremental Font Loading" OFF)
option(FT2_SUPPORT_OTVALID "Support OpenType Table Validation" OFF)
option(FT2_SUPPORT_SMOOTH "Support Anti-Aliased & Sub-Pixel Rendering" ON)
option(FT2_SUPPORT_STREAMS "Support Streamed Font Data" OFF)

cmake_dependent_option(FT2_SUPPORT_BZIP2 "Support BZip2 (.bz2) Compressed Streams" ON "BZip2_FOUND;FT2_SUPPORT_STREAMS" OFF)
cmake_dependent_option(FT2_SUPPORT_GXAAT "Support Apple GX/AAT Morphing" ON FT2_DRIVER_TRUETYPE OFF)
cmake_dependent_option(FT2_SUPPORT_GXVALID "Support GX/AAT Table Validation" OFF FT2_SUPPORT_GXAAT OFF)
cmake_dependent_option(FT2_SUPPORT_GZIP "Support GZip (.gz) Compressed Streams" ON FT2_SUPPORT_STREAMS OFF)
cmake_dependent_option(FT2_SUPPORT_LZW "Support LZW (.Z) Compressed Font Streams" ON FT2_SUPPORT_STREAMS OFF)
cmake_dependent_option(FT2_SUPPORT_PNG "Supports PNG Bitmap Font Data" OFF PNG_FOUND OFF)
cmake_dependent_option(FT2_SUPPORT_T1MM "Support Type 1 Multiple Master Fonts" ON FT2_DRIVER_TYPE1 OFF)
	
cmake_dependent_option(USE_MSVC_RUNTIME "Use MSVC Runtime Library DLL" ON MSVC OFF)

add_definitions(-DFT2_BUILD_LIBRARY)
list(APPEND FREETYPE_SOURCE
	"${PROJECT_SOURCE_DIR}/src/base/ftbase.c"
	"${PROJECT_SOURCE_DIR}/src/base/ftbbox.c"
	"${PROJECT_SOURCE_DIR}/src/base/ftbitmap.c"
	"${PROJECT_SOURCE_DIR}/src/base/ftcid.c"
	"${PROJECT_SOURCE_DIR}/src/base/ftdebug.c"
	"${PROJECT_SOURCE_DIR}/src/base/ftfntfmt.c"
	"${PROJECT_SOURCE_DIR}/src/base/ftgasp.c"
	"${PROJECT_SOURCE_DIR}/src/base/ftglyph.c"
	"${PROJECT_SOURCE_DIR}/src/base/ftinit.c"
	"${PROJECT_SOURCE_DIR}/src/base/ftpatent.c"
	"${PROJECT_SOURCE_DIR}/src/base/ftstroke.c"
	"${PROJECT_SOURCE_DIR}/src/base/ftsynth.c"
	"${PROJECT_SOURCE_DIR}/src/base/ftsystem.c"
	"${PROJECT_SOURCE_DIR}/src/autofit/autofit.c"
	"${PROJECT_SOURCE_DIR}/src/raster/raster.c")
include_directories(
	"${PROJECT_BINARY_DIR}/include"
	"${PROJECT_SOURCE_DIR}/include")

if(CMAKE_BUILD_TYPE STREQUAL "Debug")
	set(FT_DEBUG_LEVEL_ERROR ON)
	set(FT_DEBUG_LEVEL_TRACE ON)
	set(FT_DEBUG_AUTOFIT ON)
	set(FT_DEBUG_MEMORY ON)
endif()

if(FT2_DRIVER_BDF)
	set(TT_CONFIG_OPTION_BDF ON)
	list(APPEND FREETYPE_SOURCE
		"${PROJECT_SOURCE_DIR}/src/base/ftbdf.c"
		"${PROJECT_SOURCE_DIR}/src/bdf/bdf.c")
endif()

if(FT2_DRIVER_MACFNT)
	set(FT_CONFIG_OPTION_MAC_FONTS ON)
endif()

if(FT2_DRIVER_CFF)
	list(APPEND FREETYPE_SOURCE
		"${PROJECT_SOURCE_DIR}/src/cff/cff.c")
endif()

if(FT2_DRIVER_PCF)
	list(APPEND FREETYPE_SOURCE
		"${PROJECT_SOURCE_DIR}/src/pcf/pcf.c")
endif()

if(FT2_DRIVER_TRUEDOC)
	list(APPEND FREETYPE_SOURCE
		"${PROJECT_SOURCE_DIR}/src/base/ftpfr.c"
		"${PROJECT_SOURCE_DIR}/src/pfr/pfr.c")
endif()

if(FT2_DRIVER_TRUETYPE)
	list(APPEND FREETYPE_SOURCE
		"${PROJECT_SOURCE_DIR}/src/truetype/truetype.c")
endif()

if(FT2_DRIVER_TYPE1)
	list(APPEND FREETYPE_SOURCE
		"${PROJECT_SOURCE_DIR}/src/cid/type1cid.c"
		"${PROJECT_SOURCE_DIR}/src/psaux/psaux.c"
		"${PROJECT_SOURCE_DIR}/src/type1/type1.c")
endif()

if(FT2_DRIVER_WINFNT)
	list(APPEND FREETYPE_SOURCE
		"${PROJECT_SOURCE_DIR}/src/base/ftwinfnt.c"
		"${PROJECT_SOURCE_DIR}/src/winfonts/winfnt.c")
endif()

if(FT2_DRIVER_CFF OR FT2_DRIVER_TRUETYPE)
	list(APPEND FREETYPE_SOURCE
		"${PROJECT_SOURCE_DIR}/src/sfnt/sfnt.c")
endif()

if(FT2_DRIVER_CFF OR FT2_DRIVER_TYPE1)
	set(FT_CONFIG_OPTION_POSTSCRIPT_NAMES ON)
	set(FT_CONFIG_OPTION_ADOBE_GLYPH_LIST ON)
	list(APPEND FREETYPE_SOURCE
		"${PROJECT_SOURCE_DIR}/src/base/ftfstype.c"
		"${PROJECT_SOURCE_DIR}/src/base/fttype1.c"
		"${PROJECT_SOURCE_DIR}/src/pshinter/pshinter.c"
		"${PROJECT_SOURCE_DIR}/src/psnames/psnames.c")
endif()

if(FT2_DRIVER_TRUETYPE AND FT2_DRIVER_TYPE1)
	list(APPEND FREETYPE_SOURCE
		"${PROJECT_SOURCE_DIR}/src/type42/type42.c")
endif()

if(FT2_SUPPORT_BZIP2)
	set(FT_CONFIG_OPTION_USE_BZIP2 ON)
	list(APPEND FREETYPE_SOURCE
		"${PROJECT_SOURCE_DIR}/src/bzip2/ftbzip2.c")
	include_directories("${BZip2_INCLUDE_DIR}")
	list(APPEND CMAKE_REQUIRED_INCLUDES "${BZip2_INCLUDE_DIR}")
endif()

if(FT2_SUPPORT_CACHE)
	list(APPEND FREETYPE_SOURCE
		"${PROJECT_SOURCE_DIR}/src/cache/ftcache.c")
endif()

if(FT2_SUPPORT_ENVPROP)
	set(FT_CONFIG_OPTION_ENVIRONMENT_PROPERTIES ON)
endif()

if(FT2_SUPPORT_GXAAT)
	set(TT_CONFIG_OPTION_GX_VAR_SUPPORT ON)
endif()

if(FT2_SUPPORT_GXVALID)
	list(APPEND FREETYPE_SOURCE
		"${PROJECT_SOURCE_DIR}/src/base/ftgxval.c"
		"${PROJECT_SOURCE_DIR}/src/gxvalid/gxvalid.c")
endif()

if(FT2_SUPPORT_GZIP)
	set(FT_CONFIG_OPTION_USE_ZLIB ON)
	list(APPEND FREETYPE_SOURCE
		"${PROJECT_SOURCE_DIR}/src/gzip/ftgzip.c")
	if(ZLIB_FOUND)
		set(FT_CONFIG_OPTION_SYSTEM_ZLIB ON)
		include_directories("${ZLIB_INCLUDE_DIR}")
		list(APPEND CMAKE_REQUIRED_INCLUDES "${ZLIB_INCLUDE_DIR}")
	endif()
endif()

if(FT2_SUPPORT_INCREMENTAL)
	set(FT_CONFIG_OPTION_INCREMENTAL ON)
endif()

if(FT2_SUPPORT_LZW)
	set(FT_CONFIG_OPTION_USE_LZW ON)
	list(APPEND FREETYPE_SOURCE
		"${PROJECT_SOURCE_DIR}/src/lzw/ftlzw.c")
endif()

if(FT2_SUPPORT_PNG)
	set(FT_CONFIG_OPTION_USE_PNG ON)
	add_definitions(${PNG_DEFINITIONS})
	include_directories("${PNG_INCLUDE_DIR}")
	list(APPEND CMAKE_REQUIRED_INCLUDES "${PNG_INCLUDE_DIR}")
endif()

if(FT2_SUPPORT_OTVALID)
	list(APPEND FREETYPE_SOURCE
		"${PROJECT_SOURCE_DIR}/src/base/ftotval.c"
		"${PROJECT_SOURCE_DIR}/src/otvalid/otvalid.c")
endif()

if(FT2_SUPPORT_SMOOTH)
	set(FT_CONFIG_OPTION_SUBPIXEL_RENDERING ON)
	list(APPEND FREETYPE_SOURCE
		"${PROJECT_SOURCE_DIR}/src/base/ftlcdfil.c"
		"${PROJECT_SOURCE_DIR}/src/smooth/smooth.c")
endif()

if(NOT FT2_SUPPORT_STREAMS)
	set(FT_CONFIG_OPTION_DISABLE_STREAM_SUPPORT ON)
endif()

if(FT2_SUPPORT_T1MM)
	list(APPEND FREETYPE_SOURCE
		"${PROJECT_SOURCE_DIR}/src/base/ftmm.c")
else()
	set(T1_CONFIG_OPTION_NO_MM_SUPPORT ON)
endif()

#
# Compiler & Linker Options
#

if(MSVC)
	set(CMAKE_C_STANDARD_REQUIRED OFF)
	add_definitions(-D_CRT_SECURE_NO_WARNINGS)
	list(APPEND CUSTOM_CFLAGS "/W3")
	if(NOT USE_MSVC_RUNTIME)
		foreach(flag CMAKE_C_FLAGS
				CMAKE_C_FLAGS_DEBUG
				CMAKE_C_FLAGS_RELEASE
				CMAKE_C_FLAGS_MINSIZEREL
				CMAKE_C_FLAGS_RELWITHDEBINFO)
			string(REGEX REPLACE "/MDd" "/MTd" ${flag} "${${flag}}")
			string(REGEX REPLACE "/MD" "/MT" ${flag} "${${flag}}")
		endforeach()
	endif()
else()
	set(CMAKE_C_STANDARD_REQUIRED TRUE)
	
	check_c_compiler_flag("-march=native" _COMPILER_HAS_MARCH_NATIVE)
	if(_COMPILER_HAS_MARCH_NATIVE)
		list(APPEND CUSTOM_CFLAGS "-march=native")
	endif()
	check_c_compiler_flag("-Wall" _COMPILER_HAS_WARN_ALL)
	if(_COMPILER_HAS_WARN_ALL)
		list(APPEND CUSTOM_CFLAGS "-Wall")
	endif()
	check_c_compiler_flag("-Wextra" _COMPILER_HAS_WARN_EXTRA)
	if(_COMPILER_HAS_WARN_EXTRA)
		list(APPEND CUSTOM_CFLAGS "-Wextra")
	endif()
	check_c_compiler_flag("-Wno-unused-function" _COMPILER_HAS_WARN_NO_UNUSED_FUNCTION)
	if(_COMPILER_HAS_WARN_NO_UNUSED_FUNCTION)
		list(APPEND CUSTOM_CFLAGS "-Wno-unused-function")
	endif()

	set(OLD_CMAKE_REQUIRED_FLAGS ${CMAKE_REQUIRED_FLAGS})

	set(CMAKE_REQUIRED_FLAGS "-Wl,--as-needed")
	check_c_compiler_flag("" _LINKER_HAS_AS_NEEDED)
	if (_LINKER_HAS_AS_NEEDED)
		set(CUSTOM_LDFLAGS "${CUSTOM_LDFLAGS} -Wl,--as-needed")
	endif()
	set(CMAKE_REQUIRED_FLAGS "-Wl,--no-undefined")
	check_c_compiler_flag("" _LINKER_HAS_NO_UNDEFINED)
	if (_LINKER_HAS_NO_UNDEFINED)
		set(CUSTOM_LDFLAGS "${CUSTOM_LDFLAGS} -Wl,--no-undefined")
	endif()
	set(CMAKE_REQUIRED_FLAGS "-Wl,--no-allow-shlib-undefined")
	check_c_compiler_flag("" _LINKER_HAS_NO_SHLIB_UNDEFINED)
	if (_LINKER_HAS_NO_SHLIB_UNDEFINED)
		set(CUSTOM_LDFLAGS "${CUSTOM_LDFLAGS} -Wl,--no-allow-shlib-undefined")
	endif()
	set(CMAKE_REQUIRED_FLAGS "-Wl,--nxcompat")
	check_c_compiler_flag("" _LINKER_HAS_DEP)
	if (_LINKER_HAS_DEP)
		set(CUSTOM_LDFLAGS "${CUSTOM_LDFLAGS} -Wl,--nxcompat")
	endif()
	set(CMAKE_REQUIRED_FLAGS "-Wl,--dynamicbase")
	check_c_compiler_flag("" _LINKER_HAS_ASLR)
	if (_LINKER_HAS_ASLR)
		set(CUSTOM_LDFLAGS "${CUSTOM_LDFLAGS} -Wl,--dynamicbase")
	endif()
	set(CMAKE_REQUIRED_FLAGS "-Wl,--high-entropy-va")
	check_c_compiler_flag("" _LINKER_HAS_64ASLR)
	if (_LINKER_HAS_64ASLR)
		set(CUSTOM_LDFLAGS "${CUSTOM_LDFLAGS} -Wl,--high-entropy-va")
	endif()

	set(CMAKE_REQUIRED_FLAGS ${OLD_CMAKE_REQUIRED_FLAGS})
endif()

#
# Generated Files
#

check_include_file("unistd.h" HAVE_UNISTD_H)
check_include_file("fcntl.h" HAVE_FCNTL_H)
check_include_file("stdint.h" HAVE_STDINT_H)
set(TT_CONFIG_OPTION_BDF ${FT2_DRIVER_BDF})

file(MAKE_DIRECTORY "${PROJECT_BINARY_DIR}/include/freetype/config")
configure_file("${PROJECT_SOURCE_DIR}/config/ftconfig.in" "${PROJECT_BINARY_DIR}/include/freetype/config/ftconfig.h")
configure_file("${PROJECT_SOURCE_DIR}/config/ftmodule.in" "${PROJECT_BINARY_DIR}/include/freetype/config/ftmodule.h")
configure_file("${PROJECT_SOURCE_DIR}/config/ftoption.in" "${PROJECT_BINARY_DIR}/include/freetype/config/ftoption.h")

file(GLOB FT2_PUBLIC_HEADERS
	"${PROJECT_SOURCE_DIR}/include/*.h"
	"${PROJECT_SOURCE_DIR}/include/freetype/*.h"
	"${PROJECT_SOURCE_DIR}/include/freetype/config/*.h"
	"${PROJECT_BINARY_DIR}/include/freetype/config/*.h")
file(GLOB FT2_PRIVATE_HEADERS
	"${PROJECT_SOURCE_DIR}/include/freetype/internal/*.h")

#
# Build Targets
#

if(FT2_BUILD_SHARED)
	list(APPEND FT2_TARGET_TYPES "shared")
	add_library(freetype_shared SHARED
		${FT2_PUBLIC_HEADERS}
		${FT2_PRIVATE_HEADERS}
		${FREETYPE_SOURCE})
	set_target_properties(freetype_shared PROPERTIES
		OUTPUT_NAME "freetype"
		COMPILE_DEFINITIONS freetype_EXPORTS
		SOVERSION "${FT2_SOVERSION}")
	if(CUSTOM_LDFLAGS)
		set_target_properties(freetype_shared PROPERTIES
			LINK_FLAGS ${CUSTOM_LDFLAGS})
	endif()
	if(FT2_SUPPORT_BZIP2)
		target_link_libraries(freetype_shared PRIVATE ${BZip2_LIBRARIES})
	endif()
	if(FT2_SUPPORT_GZIP AND ZLIB_FOUND)
		target_link_libraries(freetype_shared PRIVATE ${ZLIB_LIBRARIES})
	endif()
	if(FT2_SUPPORT_PNG)
		target_link_libraries(freetype_shared PRIVATE ${PNG_LIBRARIES})
	endif()
	if(MSVC)
		install(FILES "${CMAKE_CURRENT_BINARY_DIR}/Debug/freetype.pdb"
			DESTINATION ${CMAKE_INSTALL_BINDIR} CONFIGURATIONS Debug)
		install(FILES "${CMAKE_CURRENT_BINARY_DIR}/RelWithDebInfo/freetype.pdb"
			DESTINATION ${CMAKE_INSTALL_BINDIR} CONFIGURATIONS RelWithDebInfo)
	endif()
endif()

if(FT2_BUILD_STATIC)
	list(APPEND FT2_TARGET_TYPES "static")
	add_library(freetype_static STATIC
		${FT2_PUBLIC_HEADERS}
		${FT2_PRIVATE_HEADERS}
		${FREETYPE_SOURCE})
	if(MSVC)
		install(FILES "${CMAKE_CURRENT_BINARY_DIR}/Debug/freetype_static.pdb"
			DESTINATION ${CMAKE_INSTALL_LIBDIR} CONFIGURATIONS Debug)
		install(FILES "${CMAKE_CURRENT_BINARY_DIR}/RelWithDebInfo/freetype_static.pdb"
			DESTINATION ${CMAKE_INSTALL_LIBDIR} CONFIGURATIONS RelWithDebInfo)
	endif()
endif()

foreach(FT2_TARGET_TYPE ${FT2_TARGET_TYPES})
	list(APPEND FT2_TARGET_BUILT "freetype_${FT2_TARGET_TYPE}")
	set_target_properties("freetype_${FT2_TARGET_TYPE}" PROPERTIES
		C_VISIBILITY_PRESET hidden
		POSITION_INDEPENDENT_CODE ON)
	if(CUSTOM_CFLAGS)
		target_compile_options("freetype_${FT2_TARGET_TYPE}" PUBLIC ${CUSTOM_CFLAGS})
	endif()
	install(TARGETS "freetype_${FT2_TARGET_TYPE}"
		RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
		LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
		ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})
endforeach()
install(FILES
	"${PROJECT_SOURCE_DIR}/LICENSE.md"
	"${PROJECT_SOURCE_DIR}/README.md"
	"${PROJECT_SOURCE_DIR}/FTL.TXT"
	DESTINATION ${CMAKE_INSTALL_DOCDIR})
install(DIRECTORY "${PROJECT_SOURCE_DIR}/include/"
	DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}"
	PATTERN "internal" EXCLUDE)
install(DIRECTORY "${PROJECT_BINARY_DIR}/include/"
	DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}")

