cmake_minimum_required(VERSION 2.8.8) 

project(UTK_Scan CXX)

#Set options depending on the C++ compiler
if (CMAKE_COMPILER_IS_GNUCXX)
  if (${CMAKE_CXX_COMPILER_VERSION} VERSION_GREATER "4.9")
    #Make compiler messages nice with colored tags.
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fdiagnostics-color=auto")
  endif(${CMAKE_CXX_COMPILER_VERSION} VERSION_GREATER "4.9")
  
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -pedantic -fPIC -std=c++0x")
endif(CMAKE_COMPILER_IS_GNUCXX)

#if user does not specify prefix we assign it to the exec directory
if (CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
  message(STATUS "Install Prefix not specified.")
  get_filename_component(INSTALL_DIR ${CMAKE_SOURCE_DIR} REALPATH)
  set(CMAKE_INSTALL_PREFIX ${INSTALL_DIR} CACHE PATH
    "Install Prefix" FORCE)
endif ()
message(STATUS "Installing to ${CMAKE_INSTALL_PREFIX}")

#Define the default build type
IF(NOT CMAKE_BUILD_TYPE)
  SET(CMAKE_BUILD_TYPE Release CACHE STRING
    "Build type, options are: None Debug Release RelWithDebInfo MinSizeRel Gprof."
    FORCE)
  message(STATUS "Build type not defined, using default: ${CMAKE_BUILD_TYPE}")
ENDIF(NOT CMAKE_BUILD_TYPE)
message(STATUS "Build type: ${CMAKE_BUILD_TYPE}")

if(CMAKE_BUILD_TYPE MATCHES "Gprof")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pg -g")
endif(CMAKE_BUILD_TYPE MATCHES "Gprof")

#Install options
option(USE_ROOT "Use ROOT for analysis" ON)
option(USE_GSL "Use GSL for Pulse Fitting" ON)
option(USE_NEWREADOUT "Use new readout format" ON)
option(USE_VERBOSE "Make Scan More Verbose" OFF)
option(USE_ONLINE "Options for Online Scans" OFF)
option(USE_DEBUG "Debugging info for TreeCorrelator" OFF)
option(USE_GGATES "Gamma-Gamma gates in GeProcessor" OFF)

if(USE_NEWREADOUT)
  add_definitions(-D newreadout)
endif(USE_NEWREADOUT)

if(USE_VERBOSE)
  add_definitions(-D VERBOSE)
endif(USE_VERBOSE)

if(USE_ONLINE)
  add_definitions(-D ONLINE)
endif(USE_ONLINE)

if(USE_DEBUG)
  add_definitions(-D DEBUG)
endif(USE_DEBUG)

if(USE_GGATES)
  add_definitions(-D GGATES)
endif(USE_GGATES)

#Find packages needed for pixie_scan
#Load additional find_package scripts.
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmakeModules/")

find_package(PixieSuite REQUIRED)
set(LIBS ${LIBS} ${PIXIE_SUITE_LIBS})

#Find ROOT if USE_ROOT was set.
if(USE_ROOT)
  find_package (ROOT REQUIRED)
  mark_as_advanced(FORCE GENREFLEX_EXECUTABLE ROOTCINT_EXECUTABLE 
    ROOT_CONFIG_EXECUTABLE)
  
  include_directories(${ROOT_INCLUDE_DIR})
  link_directories(${ROOT_LIBRARY_DIR})
  add_definitions("-D useroot")
endif(USE_ROOT)

#Check if GSL is installed 
if(USE_GSL)
  find_package(GSL REQUIRED)
  if(${GSL_VERSION} LESS 1.14)
    message(FATAL_ERROR "GSL 1.14 or higher required. Found GSL ${GSL_VERSION}")
  endif(${GSL_VERSION} LESS 1.14)
  add_definitions("-D usegsl")
endif(USE_GSL)

#Build the core library
include_directories(include)
add_subdirectory(src)
