
cmake_minimum_required(VERSION 3.17)


# Project name and a few useful settings.
project(Arktos
	VERSION 0.1
	DESCRIPTION "Orbital Manoeuvre Optimiser"
	LANGUAGES CXX)

# Only do these if this is the main project, and not if it is included through add_subdirectory
if(CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME)
  set(CMAKE_CXX_EXTENSIONS OFF)

  # Let's nicely support folders in IDEs
  set_property(GLOBAL PROPERTY USE_FOLDERS ON)
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

set(MSVC2019_COMPATIBILITY ON)
# Add Corrade as a subproject
add_subdirectory(extern/corrade EXCLUDE_FROM_ALL)

# Add Magnum as a subproject, enable Sdl2Application
if(MINGW)
    message("MINGW: added SDL to the path")
    set(CMAKE_PREFIX_PATH ${PROJECT_SOURCE_DIR}/extern/SDL2-2.0.10-mingw ${CMAKE_PREFIX_PATH})
else()
    message("MSVC: added SDL to the path")
    set(CMAKE_PREFIX_PATH ${PROJECT_SOURCE_DIR}/extern/SDL2-2.0.10-vs ${CMAKE_PREFIX_PATH})
endif(MINGW)

set(WITH_SDL2APPLICATION ON CACHE BOOL "" FORCE)
add_subdirectory(extern/magnum EXCLUDE_FROM_ALL)

set(IMGUI_DIR ${CMAKE_CURRENT_SOURCE_DIR}/extern/imgui)
set(WITH_IMGUI ON CACHE BOOL "" FORCE) # enable what you need
add_subdirectory(extern/magnum-integration EXCLUDE_FROM_ALL)

add_subdirectory(extern/spdlog)
find_package(spdlog)

SET (LUA_EXTERNALS_DIR "${PROJECT_SOURCE_DIR}/extern/lua-integration")
SET (LUA_DIR "${LUA_EXTERNALS_DIR}/lua")
SET (SOL2_DIR "${EXTERNALS_DIR}/sol")

# Lua lib
add_library(
    lua STATIC
    ${LUA_DIR}/lapi.c
    ${LUA_DIR}/lauxlib.c
    ${LUA_DIR}/lbaselib.c
    ${LUA_DIR}/lcode.c
    ${LUA_DIR}/lcorolib.c
    ${LUA_DIR}/lctype.c
    ${LUA_DIR}/ldblib.c
    ${LUA_DIR}/ldebug.c
    ${LUA_DIR}/ldo.c
    ${LUA_DIR}/ldump.c
    ${LUA_DIR}/lfunc.c
    ${LUA_DIR}/lgc.c
    ${LUA_DIR}/linit.c
    ${LUA_DIR}/liolib.c
    ${LUA_DIR}/llex.c
    ${LUA_DIR}/lmathlib.c
    ${LUA_DIR}/lmem.c
    ${LUA_DIR}/loadlib.c
    ${LUA_DIR}/lobject.c
    ${LUA_DIR}/lopcodes.c
    ${LUA_DIR}/loslib.c
    ${LUA_DIR}/lparser.c
    ${LUA_DIR}/lstate.c
    ${LUA_DIR}/lstring.c
    ${LUA_DIR}/lstrlib.c
    ${LUA_DIR}/ltable.c
    ${LUA_DIR}/ltablib.c
    ${LUA_DIR}/ltm.c
    ${LUA_DIR}/lua.c
    ${LUA_DIR}/lundump.c
    ${LUA_DIR}/lutf8lib.c
    ${LUA_DIR}/lvm.c
    ${LUA_DIR}/lzio.c)


# The compiled library code is here
add_subdirectory(src)

# Final application is here
add_subdirectory(apps)


# Testing only available if this is the main app
# Emergency override ARKTOS_BUILD_TESTING)provided as well
if((CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME OR ARKTOS_BUILD_TESTING)
   AND BUILD_TESTING)
    message("Testing enabled")
  add_subdirectory(tests)
endif()
