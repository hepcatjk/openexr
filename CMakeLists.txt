CMAKE_MINIMUM_REQUIRED ( VERSION 2.8 )

PROJECT ( openEXR )

set_property(GLOBAL PROPERTY USE_FOLDERS ON)
OPTION(OVERRIDE_OUTPUT_DIRECTORY    "Overide default folder for libraries and extecutables" OFF)



##############################################################
# Place Libs and exe files in one Output paths
##############################################################
if(NOT OVERRIDE_OUTPUT_DIRECTORY)
  SET(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}")
  SET(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}")
  SET(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}")
endif()


if (CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-switch -Wno-tautological-compare -Wno-deprecated-register")
elseif (CMAKE_CXX_COMPILER_ID MATCHES "GNU")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-unused-result")
elseif (CMAKE_CXX_COMPILER_ID MATCHES "MSVC")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /wd4275 /wd4267 /wd4244 /wd4477 /wd4996 /wd4800 /wd4334 /wd4251 /wd4305 /wd4302 /wd4311 /wd4018")
endif()



##############################################################
# Zlib Windows
##############################################################

IF(WIN32)
  # Build zlib (only on Windows)
  SET(ZLIB_BUILD_STATIC_LIBS ON CACHE BOOL " " FORCE)
  SET(ZLIB_BUILD_SHARED_LIBS OFF CACHE BOOL " " FORCE)
  ADD_SUBDIRECTORY(zlib)
  SET(ZLIB_INCLUDE_DIR "${CMAKE_SOURCE_DIR}/zlib")
  if(MSVC)
  set_property(TARGET zlibstatic PROPERTY DEBUG_POSTFIX "") # turn off the d trailer on library name, too messy for import in openexr.
  endif()
  SET(ZLIB_LIBRARY "${CMAKE_BINARY_DIR}/$(CONFIGURATION)/zlibstatic.lib")
  SET(ZLIB_ROOT "${CMAKE_BINARY_DIR}/$(CONFIGURATION)")
  message(STATUS "ZLIB LIBRARY: " ${ZLIB_LIBRARY})
  message(STATUS "ZLIB ROOT: " ${ZLIB_ROOT})
  SET_PROPERTY(TARGET zlibstatic zlib minigzip example PROPERTY FOLDER "zlib")
  INCLUDE_DIRECTORIES(${ZLIB_INCLUDE_DIR} "${CMAKE_CURRENT_SOURCE_DIR}/zlib")
  add_definitions(-DWIN32)
ENDIF()


##############################################################
# ILMBase
##############################################################
OPTION (NAMESPACE_VERSIONING "Namespace Versioning" OFF)
OPTION (BUILD_SHARED_LIBS "Namespace Versioning" OFF)
ADD_SUBDIRECTORY(openexr/IlmBase)
SET_PROPERTY(TARGET Iex IexMath Half Imath IlmThread HalfTest IexTest IlmThread ImathTest toFloat eLut PROPERTY FOLDER "ILMBase")


##############################################################
# OpenEXR
##############################################################
include_directories(
  openexr/IlmBase/Iex
  openexr/IlmBase/Imath
  openexr/IlmBase/IlmThread
  openexr/IlmBase/Half
  openexr/IlmBase/config
  openexr/IlmBase/Imath
  openexr/OpenEXR/config
  openexr/IlmBase/config
  ${CMAKE_CURRENT_SOURCE_DIR}/zlib
  ${CMAKE_CURRENT_BINARY_DIR}/zlib
)


IF(WIN32)
include_directories(
  openexr/IlmBase/config.windows
  )
ELSE()
include_directories(
  ${CMAKE_BINARY_DIR}/openexr/OpenEXR/config 
  ${CMAKE_BINARY_DIR}/openexr/IlmBase/config
  )
ENDIF()

 IF(WIN32)
   SET(OPENEXR_LIBS IlmImf Imath Half zlibstatic)
 ELSE()
   SET(OPENEXR_LIBS IlmImf Imath Half)
 ENDIF()

SET(EXR_LIBS Iex IlmImf IlmThread)
SET(ILMBASE_PACKAGE_PREFIX "${CMAKE_BINARY_DIR}/$(CONFIGURATION)")

LINK_DIRECTORIES ("${CMAKE_BINARY_DIR}")
SET(BUILD_SHARED_LIBS OFF CACHE BOOL " " FORCE)
SET(NAMESPACE_VERSIONING OFF CACHE BOOL " " FORCE)

ADD_SUBDIRECTORY(openexr/OpenEXR)
SET_PROPERTY(TARGET b44ExpLogTable dwaLookups IlmImfUtilTest IlmImfUtil exrenvmap exrheader IlmImfTest exrmakepreview exrmaketiled exrmultipart exrmultiview exrstdattr IlmImf IlmImfExamples IlmImfFuzzTest PROPERTY FOLDER "openEXR")


IF(WIN32)
add_custom_target(CopyIlmBaseLibs
  COMMAND ${CMAKE_COMMAND} -E copy
  ${CMAKE_BINARY_DIR}/$(CONFIGURATION)/b44ExpLogTable.exe
  ${CMAKE_CURRENT_BINARY_DIR}/openexr/OpenEXR/IlmImf/$(CONFIGURATION)/b44ExpLogTable.exe
   COMMAND ${CMAKE_COMMAND} -E copy
  ${CMAKE_BINARY_DIR}/$(CONFIGURATION)/dwaLookups.exe
  ${CMAKE_CURRENT_BINARY_DIR}/openexr/OpenEXR/IlmImf/$(CONFIGURATION)/dwaLookups.exe 
)
ELSE()
add_custom_target(CopyIlmBaseLibs
  COMMAND ${CMAKE_COMMAND} -E copy
  ${CMAKE_BINARY_DIR}/b44ExpLogTable
  ${CMAKE_CURRENT_BINARY_DIR}/openexr/OpenEXR/IlmImf/b44ExpLogTable
   COMMAND ${CMAKE_COMMAND} -E copy
  ${CMAKE_BINARY_DIR}/dwaLookups
  ${CMAKE_CURRENT_BINARY_DIR}/openexr/OpenEXR/IlmImf/dwaLookups
)
ENDIF()
SET_PROPERTY(TARGET CopyIlmBaseLibs b44ExpLogTable dwaLookups IlmImfUtilTest IlmImfUtil exrenvmap exrheader IlmImfTest exrmakepreview exrmaketiled exrmultipart exrmultiview exrstdattr IlmImf IlmImfExamples IlmImfFuzzTest PROPERTY FOLDER "openEXR")

add_dependencies(CopyIlmBaseLibs Iex Imath IlmThread Half)
add_dependencies(CopyIlmBaseLibs b44ExpLogTable dwaLookups)
add_dependencies(IlmImf CopyIlmBaseLibs)
