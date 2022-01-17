cmake_minimum_required(VERSION 3.5)
project(libuvcpp
  VERSION 0.0.6
  LANGUAGES C CXX
)

# Default to C99
if(NOT CMAKE_C_STANDARD)
  set(CMAKE_C_STANDARD 99)
endif()

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

if(NOT CMAKE_BUILD_TARGET)
    message(STATUS "No target type selected, default to both shared and static library")
    set(CMAKE_BUILD_TARGET "Both" CACHE STRING "" FORCE)
endif()

option(BUILD_EXAMPLE "Build example program" ON)
option(BUILD_TEST "Build test program" OFF)
option(ENABLE_UVC_DEBUGGING "Enable UVC debugging" OFF)

set(libuvcpp_DESCRIPTION "A cross-platform wrapper library around libuvc")
set(libuvcpp_URL "https://github.com/JWhitleyWork/libuvcpp")

set(SOURCES
#  src/ctrl.c
#  src/ctrl-gen.c
#  src/device.c
#  src/diag.c
#  src/frame.c
  src/init.cpp
#  src/stream.c
#  src/misc.c
)

find_package(libuvc REQUIRED)

if(${CMAKE_BUILD_TARGET} MATCHES "Shared")
  set(BUILD_UVCPP_SHARED TRUE)
elseif(${CMAKE_BUILD_TARGET} MATCHES "Static")
  set(BUILD_UVCPP_STATIC TRUE)
elseif(${CMAKE_BUILD_TARGET} MATCHES "Both")
  set(BUILD_UVCPP_SHARED TRUE)
  set(BUILD_UVCPP_STATIC TRUE)
else()
  message(FATAL_ERROR "Invalid build type ${CMAKE_BUILD_TARGET}" )
endif()

if(BUILD_UVCPP_SHARED)
  add_library(uvcpp SHARED ${SOURCES})
  set_target_properties(uvcpp PROPERTIES
    VERSION ${libuvcpp_VERSION}
    SOVERSION ${libuvcpp_VERSION_MAJOR}
    # Exported name of target within namespace LibUVC
    EXPORT_NAME UVCPPShared
  )
  list(APPEND UVCPP_TARGETS uvcpp)
  if(NOT LibUVCPP_STATIC)
    add_library(LibUVCPP::UVCPP ALIAS uvcpp)
  endif()
  # Aliases defined here available only if project included
  # via addsubdirectory
  add_library(LibUVCPP::UVCPPShared ALIAS uvcpp)
endif()

if(BUILD_UVCPP_STATIC)
  add_library(uvcpp_static STATIC ${SOURCES})
  set_target_properties(uvcpp_static PROPERTIES
    OUTPUT_NAME uvcpp
    # Exported name of target within namespace LibUVC
    EXPORT_NAME UVCPPStatic
  )
  list(APPEND UVCPP_TARGETS uvcpp_static)
  add_library(LibUVCPP::UVCPPStatic ALIAS uvcpp_static)
  if(LibUVCPP_STATIC)
    add_library(LibUVCPP::UVCPP ALIAS uvcpp_static)
  endif()
endif()

foreach(target_name IN LISTS UVCPP_TARGETS)
  target_include_directories(${target_name}
    PUBLIC
      # Different paths for includes for build and install phase supported
      # via INSTALL_INTERFACE and BUILD_INTERFACE generator expressions.
      $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/include>
      $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/include>
      $<INSTALL_INTERFACE:include>
  )
  target_link_libraries(${target_name}
    PUBLIC uvc
  )
  set_target_properties(${target_name} PROPERTIES
      PUBLIC_HEADER "include/libuvcpp/libuvcpp.hpp"
  )
  if(ENABLE_UVCPP_DEBUGGING)
    target_compile_definitions(${target_name}
      PRIVATE
        UVCPP_DEBUGGING
    )
  endif()
endforeach()

set(CMAKE_INSTALL_CMAKEDIR ${CMAKE_INSTALL_LIBDIR}/cmake/libuvcpp)

install(
  TARGETS ${UVCPP_TARGETS}
  EXPORT libuvcppTargets
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
  PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/libuvcpp
)

install(EXPORT libuvcppTargets
  FILE libuvcppTargets.cmake
  NAMESPACE LibUVCPP::
  DESTINATION ${CMAKE_INSTALL_CMAKEDIR}
)

include(CMakePackageConfigHelpers)
write_basic_package_version_file(libuvcppConfigVersion.cmake
  COMPATIBILITY AnyNewerVersion
)

install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/libuvcppConfigVersion.cmake
  DESTINATION ${CMAKE_INSTALL_CMAKEDIR}
)

configure_file(libuvcpp.pc.in
  ${PROJECT_BINARY_DIR}/libuvcpp.pc
  @ONLY
)

install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/libuvcpp.pc
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig
)

install(FILES libuvcppConfig.cmake
  DESTINATION ${CMAKE_INSTALL_CMAKEDIR}
)
