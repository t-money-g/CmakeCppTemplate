cmake_minimum_required( VERSION 3.11...3.22)


#cmake basics recommendations 
if(${CMAKE_VERSION} VERSION_LESS 3.14)
	 cmake_policy(VERSION ${CMAKE_MAJOR_VERSION}.${CMAKE_MINOR_VERSION})
endif()

# For a new project it is sufficient to change only its name in the following line
set( PROJECT_NAME EmptyPro )

project( ${PROJECT_NAME}
		 VERSION 1.0 
		 LANGUAGES CXX)


# Only do these if this is the main project, and not if it is included through add_subdirectory
if(CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME)

  # Optionally set things like CMAKE_CXX_STANDARD, CMAKE_POSITION_INDEPENDENT_CODE here

  # Let's ensure -std=c++xx instead of -std=g++xx
  set(CMAKE_CXX_EXTENSIONS OFF)

  # Let's nicely support folders in IDEs
  set_property(GLOBAL PROPERTY USE_FOLDERS ON)

  # Testing only available if this is the main app
  # Note this needs to be done in the main CMakeLists
  # since it calls enable_testing, which must be in the
  # main CMakeLists.
  include(CTest)

  # Docs only available if this is the main app
  find_package(Doxygen)
  if(Doxygen_FOUND)
    add_subdirectory(docs)
  else()
    message(STATUS "Doxygen not found, not building docs")
  endif()
endif()

# FetchContent added in CMake 3.11, downloads during the configure step
include(FetchContent)
# FetchContent_MakeAvailable was not added until CMake 3.14; use our shim
if(${CMAKE_VERSION} VERSION_LESS 3.14)
  include(cmake/add_FetchContent_MakeAvailable.cmake)
endif()

# Accumulator library
# This is header only, so could be replaced with git submodules or FetchContent
find_package(Boost REQUIRED)
# Adds Boost::boost


# Formatting library
FetchContent_Declare(
  fmtlib
  GIT_REPOSITORY https://github.com/fmtlib/fmt.git
  GIT_TAG 5.3.0)
FetchContent_MakeAvailable(fmtlib)
# Adds fmt::fmt

#compiled code is here 
add_subdirectory(src)

# the exes are here 
add_subdirectory(apps) 

# Testing only available if this is the main app
# Emergency override MODERN_CMAKE_BUILD_TESTING provided as well
if((CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME OR MODERN_CMAKE_BUILD_TESTING)
   AND BUILD_TESTING)
  add_subdirectory(tests)
endif()

#get submodules 
find_package(Git QUIET)

if(GIT_FOUND AND EXISTS "${PROJECT_SOURCE_DIR}/.git")
    execute_process(COMMAND ${GIT_EXECUTABLE} submodule update --init --recursive
                    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
                    RESULT_VARIABLE GIT_SUBMOD_RESULT)
    if(NOT GIT_SUBMOD_RESULT EQUAL "0")
        message(FATAL_ERROR "git submodule update --init --recursive failed with ${GIT_SUBMOD_RESULT}, please checkout submodules")
    endif()
endif()

#set( CMAKE_BUILD_TYPE Debug )
#set( CMAKE_BUILD_TYPE Release )

#[[ADD_DEFINITIONS(-g++ -O2 -fsigned-char -freg-struct-return -Wall -W -Wshadow -Wstrict-prototypes -Wpointer-arith -Wcast-qual -Winline -Werror)]]

#if( WIN32 )
#	set( CMAKE_CXX_FLAGS "/DWIN32 /D_WINDOWS /W3 /GR /EHsc /std:c++17 /D_UNICODE /DUNICODE" )
#	set( CMAKE_CXX_FLAGS_DEBUG "/MDd /Zi /Ob0 /Od /RTC1 /std:c++17 /D_UNICODE /DUNICODE" )
#	message( "Win settings chosen..." )
#elseif( ${CMAKE_SYSTEM_NAME} STREQUAL "Darwin" )
#	set( CMAKE_CXX_FLAGS "-std=c++17 -Wall" )
	#set( CMAKE_CXX_FLAGS_DEBUG "-g -std=c++17 -Wall" )
	#message( "Mac settings chosen..." )
#elseif( UNIX )
#	set( CMAKE_CXX_FLAGS "-std=c++17 -Wall" )
#	set( CMAKE_CXX_FLAGS_DEBUG "-g -std=c++17 -Wall" )
#	message( "Linux settings chosen..." )
#endif()


# Inform CMake where the header files are
#include_directories( include )

#[[INCLUDE(FetchContent)
FetchContent_Declare(
		Catch2
		GIT_REPOSITORY https://github.com/catchorg/Catch2.git
		GIT_TAG			v2.13.1)
#FetchContent_MakeAvailable(Catch2)
]]

# Automatically add all *.cpp and *.h files to the project
#file ( GLOB SOURCES "./src/*.cpp" "./include/*.h" )
#add_executable( ${PROJECT_NAME} ${SOURCES})

#target_link_libraries( ${PROJECT_NAME} Catch2::Catch2)

# Set the default project 
#set_property( DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT ${PROJECT_NAME} )


#message( "CMAKE_BUILD_TYPE is ${CMAKE_BUILD_TYPE}" )
