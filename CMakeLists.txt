cmake_minimum_required(VERSION 3.14)
project(lanelet2_vendor LANGUAGES CXX)

set(CMAKE_POSITION_INDEPENDENT_CODE ON)

option(LANELET2_SHARED_LIBS "Build lanelet2 as shared libraries (.so)" ON)
option(LANELET2_BUILD_CORE "Build lanelet2_core (required)" ON)
option(LANELET2_BUILD_IO "Build lanelet2_io (required)" ON)
option(LANELET2_BUILD_PROJECTION "Build lanelet2_projection (required)" ON)
option(LANELET2_BUILD_TRAFFIC_RULES "Build lanelet2_traffic_rules (required)" ON)
option(LANELET2_BUILD_ROUTING "Build lanelet2_routing (required)" ON)
option(LANELET2_BUILD_MATCHING "Build lanelet2_matching" OFF)
option(LANELET2_BUILD_VALIDATION "Build lanelet2_validation" OFF)
option(LANELET2_BUILD_MAPS "Build/install lanelet2_maps assets" OFF)
option(LANELET2_BUILD_EXAMPLES "Build lanelet2_examples executables" OFF)
option(LANELET2_BUILD_PYTHON "Build lanelet2_python bindings" OFF)

if(NOT LANELET2_BUILD_CORE
        OR NOT LANELET2_BUILD_IO
        OR NOT LANELET2_BUILD_PROJECTION
        OR NOT LANELET2_BUILD_TRAFFIC_RULES
        OR NOT LANELET2_BUILD_ROUTING)
    message(
            FATAL_ERROR
            "lanelet2_vendor requires lanelet2_core, lanelet2_io, lanelet2_projection, lanelet2_traffic_rules and lanelet2_routing to be enabled.")
endif()

if(LANELET2_BUILD_PYTHON AND NOT LANELET2_BUILD_MATCHING)
    message(FATAL_ERROR "lanelet2_python requires LANELET2_BUILD_MATCHING=ON.")
endif()

if(LANELET2_SHARED_LIBS)
    set(LANELET2_LIB_TYPE SHARED)
else()
    set(LANELET2_LIB_TYPE STATIC)
endif()

set(LANELET2_SUPPRESS_WARNINGS
        -Wno-deprecated-declarations
        -Wno-unused-parameter
        -Wno-sign-compare)

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

if(LANELET2_BUILD_CORE)
    add_subdirectory(lanelet2_core)
endif()

if(LANELET2_BUILD_IO)
    add_subdirectory(lanelet2_io)
endif()

if(LANELET2_BUILD_PROJECTION)
    add_subdirectory(lanelet2_projection)
endif()

if(LANELET2_BUILD_TRAFFIC_RULES)
    add_subdirectory(lanelet2_traffic_rules)
endif()

if(LANELET2_BUILD_ROUTING)
    add_subdirectory(lanelet2_routing)
endif()

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

message(STATUS "lanelet2: modular plain CMake build (ROS/mrt free)")
