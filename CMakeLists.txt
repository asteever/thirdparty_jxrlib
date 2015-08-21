cmake_minimum_required(VERSION 2.8)
project(jxrlib)
set(jxrlib_VERSION 1.1.0)
set(jxrlib_VERSION 1.1.0 PARENT_SCOPE)
set(jxrlib_SOVERSION 1)
 
option(JXR_BUILD_SHARED "Build jxrlib shared libraries" OFF)
option(JXR_BUILD_STATIC "Build jxrlib static libraries" ON)
option(JXR_BUILD_EXES "Build jxrlib executables (requires jxrlib shared libraries)" OFF)
option(JXR_INSTALL "Install jxrlib libraries and executables" OFF)

set(bindir bin)
set(incdir include)
set(libdir lib)
 
set(CMAKE_SKIP_BUILD_RPATH OFF)
set(CMAKE_BUILD_WITH_INSTALL_RPATH OFF)
set(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/${libdir}")
set(CMAKE_INSTALL_NAME_DIR "${CMAKE_INSTALL_PREFIX}/${libdir}")
 
add_definitions(-DDISABLE_PERF_MEASUREMENT)
if(NOT MSVC)
	# Inhibit all warnings with -w; that's in the Makefile provided
	# with jxrlib, and there are lots of warnings otherwise.
	add_definitions(-D__ANSI__ -w -std=c99 -fPIC)
endif()
include_directories(common/include image/sys jxrgluelib jxrtestlib)
set(jxrlib_INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/common/include ${CMAKE_CURRENT_SOURCE_DIR}/image/sys ${CMAKE_CURRENT_SOURCE_DIR}/jxrgluelib ${CMAKE_CURRENT_SOURCE_DIR}/jxrtestlib PARENT_SCOPE)
 
# sources
file(GLOB SRC_DEC "image/decode/*.c")
file(GLOB SRC_ENC "image/encode/*.c")
file(GLOB SRC_SYS "image/sys/*.c")
file(GLOB SRC_GLUE "jxrgluelib/*.c")
file(GLOB SRC_TEST "jxrtestlib/*.c")
 
# libraries
if(JXR_BUILD_SHARED)
	add_library(libjpegxr_shared SHARED ${SRC_SYS} ${SRC_ENC} ${SRC_DEC})
	add_library(libjxrglue_shared SHARED ${SRC_GLUE} ${SRC_TEST})
	target_link_libraries(libjxrglue_shared m libjpegxr_shared)
	set(jxrlib_SHARED_LIBRARIES libjxrglue_shared libjpegxr_shared)
	set(jxrlib_SHARED_LIBRARIES libjxrglue_shared libjpegxr_shared PARENT_SCOPE)
endif()
if(JXR_BUILD_STATIC)
	add_library(libjpegxr_static STATIC ${SRC_SYS} ${SRC_ENC} ${SRC_DEC})
	add_library(libjxrglue_static STATIC ${SRC_GLUE} ${SRC_TEST})
	set(jxrlib_STATIC_LIBRARIES libjxrglue_static libjpegxr_static)
	set(jxrlib_STATIC_LIBRARIES libjxrglue_static libjpegxr_static PARENT_SCOPE)
endif()
 
# executables
if(JXR_BUILD_EXES)
	add_executable(JxrEncApp jxrencoderdecoder/JxrEncApp.c)
	add_executable(JxrDecApp jxrencoderdecoder/JxrDecApp.c)
	target_link_libraries(JxrEncApp m ${jxrlib_SHARED_LIBRARIES})
	target_link_libraries(JxrDecApp m ${jxrlib_SHARED_LIBRARIES})
	set(jxrlib_EXECUTABLES JxrEncApp JxrDecApp)
endif()
 
# set names and versions of libraries
if(JXR_BUILD_SHARED)
	set_target_properties(libjpegxr_shared PROPERTIES
		OUTPUT_NAME jpegxr
		VERSION ${jxrlib_VERSION}
		SOVERSION ${jxrlib_SOVERSION}
	)
	set_target_properties(libjxrglue_shared PROPERTIES
		OUTPUT_NAME jxrglue
		VERSION ${jxrlib_VERSION}
		SOVERSION ${jxrlib_SOVERSION}
	)
endif()
if(JXR_BUILD_STATIC)
	set_target_properties(libjpegxr_static PROPERTIES
		OUTPUT_NAME jpegxr
	)
	set_target_properties(libjxrglue_static PROPERTIES
		OUTPUT_NAME jxrglue
	)
endif()
 
# install
if(JXR_INSTALL)
	install(TARGETS
		${jxrlib_EXECUTABLES}
		${jxrlib_SHARED_LIBRARIES}
		${jxrlib_STATIC_LIBRARIES}
		RUNTIME DESTINATION ${bindir}
		LIBRARY DESTINATION ${libdir}
		ARCHIVE DESTINATION ${libdir}
	)
 
	file(GLOB HEADERS_DEC "image/decode/*.h")
	file(GLOB HEADERS_ENC "image/encode/*.h")
	file(GLOB HEADERS_SYS "image/sys/*.h")
	file(GLOB HEADERS_GLUE "jxrgluelib/*.h")
	file(GLOB HEADERS_TEST "jxrtestlib/*.h")
 
	install(FILES ${HEADERS_DEC} ${HEADERS_ENC} ${HEADERS_SYS}
	  DESTINATION "${incdir}/jxrlib"
	)
	install(FILES ${HEADERS_GLUE} ${HEADERS_TEST}
	  DESTINATION "${incdir}/jxrgluelib"
	)
endif()
