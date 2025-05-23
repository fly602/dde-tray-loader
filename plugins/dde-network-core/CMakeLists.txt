# SPDX-FileCopyrightText: 2024 UnionTech Software Technology Co., Ltd.
#
# SPDX-License-Identifier: CC0-1.0

cmake_minimum_required(VERSION 3.7)

project(dde-network-core)

set(VERSION "1.0.0" CACHE STRING "define project version")

# 增加安全编译参数
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fstack-protector-all")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fstack-protector-all")
set(CMAKE_EXE_LINKER_FLAGS  "${CMAKE_EXE_LINKER_FLAGS} -z relro -z now -z noexecstack -pie")

set(DDE-Network-Core_INCLUDE_DIRS  "${CMAKE_SOURCE_DIR}/src")
set(DDE-Network-Core_LIBRARIES  dde-network-core)

find_package(PkgConfig REQUIRED)
find_package(KF6NetworkManagerQt REQUIRED)
pkg_check_modules(LibNM REQUIRED IMPORTED_TARGET libnm)
include_directories(${LibNM_INCLUDE_DIRS})

pkg_check_modules(Gio REQUIRED gio-2.0)
include_directories(${Gio_INCLUDE_DIRS})

pkg_search_module(GLIB REQUIRED glib-2.0)
include_directories(${GLIB_INCLUDE_DIRS})
link_directories(${GLIB_LIBRARY_DIRS})

set(TRY_KF6_LIBRARIES
IMPORTED_LOCATION_DEBIAN
IMPORTED_LOCATION_NOCONFIG
)

if (NOT KF6_QT_LIBRARIES)
    find_library(KF6_QT_LIBRARIES libKF6NetworkManagerQt.so)
endif()

find_package(Qt${QT_VERSION_MAJOR} ${REQUIRED_QT_VERSION} COMPONENTS Core Widgets DBus Network LinguistTools REQUIRED)
get_target_property(KF6_QT_INCLUDE_DIRS KF6::NetworkManagerQt INTERFACE_INCLUDE_DIRECTORIES)

# test networkmanagetqt
try_compile(DEEPIN_NMQT_TEST
              ${CMAKE_CURRENT_BINARY_DIR}/try_compile
            SOURCES
              ${CMAKE_CURRENT_SOURCE_DIR}/try-compile/networkmanagerqt.cc
            LINK_LIBRARIES
              ${KF6_QT_LIBRARIES}
              Qt${QT_VERSION_MAJOR}::Core
              Qt${QT_VERSION_MAJOR}::DBus
              Qt${QT_VERSION_MAJOR}::Network
            CMAKE_FLAGS
              "-DINCLUDE_DIRECTORIES=${KF6_QT_INCLUDE_DIRS}\;${GLIB_INCLUDE_DIRS}\;${LibNM_INCLUDE_DIRS}"
            OUTPUT_VARIABLE
              result
)

if (QT_DEBUG)
    message(" KF6_QT_INCLUDE_DIRS : " ${KF6_QT_INCLUDE_DIRS})
    message(" GLIB_INCLUDE_DIRS :" ${GLIB_INCLUDE_DIRS})
    message(" CMAKE_CURRENT_SOURCE_DIR: " ${CMAKE_CURRENT_SOURCE_DIR})
    message(STATUS ${result})
endif()

if (DEEPIN_NMQT_TEST)
    add_definitions(-DUSE_DEEPIN_NMQT)
endif()

message(STATUS DEEPIN_NMQT_TEST ": ${DEEPIN_NMQT_TEST}")

set(INTERFACE_VERSION "1.6")
add_definitions(-DINTERFACE_VERSION="${INTERFACE_VERSION}")
message("Interface Version: " ${INTERFACE_VERSION})
set(NET_USE_NOTIFICATION ON)
add_definitions(-DNET_SUPPORT_WPAEAPSUITEB192)

if (CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
    set(CMAKE_INSTALL_PREFIX "/usr")
endif()

if (NET_USE_NOTIFICATION)
    add_definitions(-DNET_USE_NOTIFICATION)
endif()

include_directories(src)

add_subdirectory("src")
add_subdirectory("dock-tray-network-plugin")
