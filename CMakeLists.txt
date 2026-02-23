cmake_minimum_required(VERSION 3.14)
project(lanelet2_vendor LANGUAGES CXX)

set(CMAKE_POSITION_INDEPENDENT_CODE ON)

option(LANELET2_SHARED_LIBS "Build lanelet2 as shared libraries (.so)" ON)
option(LANELET2_BUILD_MATCHING "Build lanelet2_matching" OFF)
option(LANELET2_BUILD_VALIDATION "Build lanelet2_validation" OFF)
option(LANELET2_BUILD_MAPS "Build/install lanelet2_maps assets" OFF)
option(LANELET2_BUILD_EXAMPLES "Build lanelet2_examples executables" OFF)
option(LANELET2_BUILD_PYTHON "Build lanelet2_python bindings" OFF)

if(LANELET2_SHARED_LIBS)
    set(LANELET2_LIB_TYPE SHARED)
else()
    set(LANELET2_LIB_TYPE STATIC)
endif()

set(LANELET2_SUPPRESS_WARNINGS
        -Wno-deprecated-declarations
        -Wno-unused-parameter
        -Wno-sign-compare)
set(LANELET2_EXPORT_SET lanelet2Targets)

set(_lanelet2_boost_components filesystem serialization)
if(LANELET2_BUILD_VALIDATION)
    list(APPEND _lanelet2_boost_components program_options)
endif()
list(REMOVE_DUPLICATES _lanelet2_boost_components)

find_package(Boost REQUIRED COMPONENTS ${_lanelet2_boost_components})
find_package(Eigen3 REQUIRED)
find_package(pugixml REQUIRED)
if(NOT TARGET pugixml::pugixml AND TARGET pugixml)
    add_library(pugixml::pugixml ALIAS pugixml)
endif()

list(APPEND CMAKE_MODULE_PATH "/usr/share/cmake/geographiclib")
find_package(GeographicLib REQUIRED)

if(TARGET GeographicLib::GeographicLib)
    set(LANELET2_GEOGRAPHICLIB_TARGET GeographicLib::GeographicLib)
    unset(LANELET2_GEOGRAPHICLIB_INCLUDE_DIRS)
else()
    set(LANELET2_GEOGRAPHICLIB_TARGET ${GeographicLib_LIBRARIES})
    set(LANELET2_GEOGRAPHICLIB_INCLUDE_DIRS ${GeographicLib_INCLUDE_DIRS})
endif()

add_subdirectory(lanelet2_core)
add_subdirectory(lanelet2_io)
add_subdirectory(lanelet2_projection)
add_subdirectory(lanelet2_traffic_rules)
add_subdirectory(lanelet2_routing)

if(LANELET2_BUILD_MATCHING)
    add_subdirectory(lanelet2_matching)
endif()

if(LANELET2_BUILD_MAPS)
    add_subdirectory(lanelet2_maps)
endif()

if(LANELET2_BUILD_VALIDATION)
    add_subdirectory(lanelet2_validation)
endif()

if(LANELET2_BUILD_EXAMPLES)
    add_subdirectory(lanelet2_examples)
endif()

if(LANELET2_BUILD_PYTHON)
    add_subdirectory(lanelet2_python)
endif()

include(GNUInstallDirs)
install(
        EXPORT ${LANELET2_EXPORT_SET}
        NAMESPACE lanelet2_
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/lanelet2)

message(STATUS "lanelet2: modular plain CMake build (ROS/mrt free)")
