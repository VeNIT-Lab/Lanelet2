# =============================================================================
# CMakeLists.txt - Build lanelet2 packages WITHOUT ROS/mrt_cmake
# =============================================================================
#
# Replaces the mrt_cmake_modules build system (which depends on ROS) with
# plain CMake. The lanelet2 C++ code itself has ZERO ROS dependencies.
#
# Usage in your root CMakeLists.txt:
#   add_subdirectory(extern/Lanelet2)
#   target_link_libraries(your_target PUBLIC lanelet2_core)
#
# Available targets:
#   lanelet2_core           - Core primitives, geometry, LaneletMap
#   lanelet2_io             - OSM/binary map reading and writing
#   lanelet2_projection     - WGS84 <-> local coordinate projections
#   lanelet2_traffic_rules  - Traffic rule interpretation
#   lanelet2_routing        - Routing graph, shortest path, reachable sets
#
# Prerequisites:
#   sudo apt install libboost-dev libeigen3-dev libpugixml-dev libgeographiclib-dev
# =============================================================================

cmake_minimum_required(VERSION 3.14)

set(LANELET2_DIR "${CMAKE_CURRENT_SOURCE_DIR}")

# Common warning suppression for thirdparty code
set(LANELET2_SUPPRESS_WARNINGS -Wno-deprecated-declarations -Wno-unused-parameter -Wno-sign-compare)

# PIC is always needed (shared libs require it, static libs need it if linked into .so)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

# Option: build as shared or static libraries
# Set LANELET2_SHARED_LIBS=ON to produce .so instead of .a
option(LANELET2_SHARED_LIBS "Build lanelet2 as shared libraries (.so)" ON)
if(LANELET2_SHARED_LIBS)
    set(LANELET2_LIB_TYPE SHARED)
else()
    set(LANELET2_LIB_TYPE STATIC)
endif()

# =============================================================================
# Dependencies
# =============================================================================

find_package(Boost REQUIRED COMPONENTS filesystem serialization)
find_package(Eigen3 REQUIRED)
find_package(pugixml REQUIRED)

# Focal's pugixml (1.10) exports a plain "pugixml" target, not the
# namespaced "pugixml::pugixml" that newer versions provide.
if(NOT TARGET pugixml::pugixml)
    add_library(pugixml::pugixml ALIAS pugixml)
endif()

list(APPEND CMAKE_MODULE_PATH "/usr/share/cmake/geographiclib")
find_package(GeographicLib REQUIRED)

# =============================================================================
# lanelet2_core
# =============================================================================

file(GLOB LANELET2_CORE_SRC "${LANELET2_DIR}/lanelet2_core/src/*.cpp")
add_library(lanelet2_core ${LANELET2_LIB_TYPE} ${LANELET2_CORE_SRC})
target_include_directories(lanelet2_core PUBLIC
        $<BUILD_INTERFACE:${LANELET2_DIR}/lanelet2_core/include>
        $<INSTALL_INTERFACE:include>
)
target_link_libraries(lanelet2_core PUBLIC Boost::boost Eigen3::Eigen)
target_compile_features(lanelet2_core PUBLIC cxx_std_14)
target_compile_options(lanelet2_core PRIVATE ${LANELET2_SUPPRESS_WARNINGS})

# =============================================================================
# lanelet2_io
# =============================================================================

file(GLOB LANELET2_IO_SRC "${LANELET2_DIR}/lanelet2_io/src/*.cpp")
add_library(lanelet2_io ${LANELET2_LIB_TYPE} ${LANELET2_IO_SRC})
target_include_directories(lanelet2_io PUBLIC
        $<BUILD_INTERFACE:${LANELET2_DIR}/lanelet2_io/include>
        $<INSTALL_INTERFACE:include>
)
target_link_libraries(lanelet2_io
        PUBLIC  lanelet2_core Boost::filesystem Boost::serialization
        PRIVATE pugixml::pugixml
)
target_compile_options(lanelet2_io PRIVATE ${LANELET2_SUPPRESS_WARNINGS})

# =============================================================================
# lanelet2_projection
# =============================================================================

file(GLOB LANELET2_PROJECTION_SRC "${LANELET2_DIR}/lanelet2_projection/src/*.cpp")
add_library(lanelet2_projection ${LANELET2_LIB_TYPE} ${LANELET2_PROJECTION_SRC})
target_include_directories(lanelet2_projection PUBLIC
        $<BUILD_INTERFACE:${LANELET2_DIR}/lanelet2_projection/include>
        $<INSTALL_INTERFACE:include>
)
target_include_directories(lanelet2_projection PRIVATE ${GeographicLib_INCLUDE_DIRS})
target_link_libraries(lanelet2_projection
        PUBLIC  lanelet2_core lanelet2_io
        PRIVATE ${GeographicLib_LIBRARIES}
)
target_compile_options(lanelet2_projection PRIVATE ${LANELET2_SUPPRESS_WARNINGS})

# =============================================================================
# lanelet2_traffic_rules
# =============================================================================

file(GLOB LANELET2_TRAFFIC_RULES_SRC "${LANELET2_DIR}/lanelet2_traffic_rules/src/*.cpp")
add_library(lanelet2_traffic_rules ${LANELET2_LIB_TYPE} ${LANELET2_TRAFFIC_RULES_SRC})
target_include_directories(lanelet2_traffic_rules PUBLIC
        $<BUILD_INTERFACE:${LANELET2_DIR}/lanelet2_traffic_rules/include>
        $<INSTALL_INTERFACE:include>
)
target_link_libraries(lanelet2_traffic_rules PUBLIC lanelet2_core)
target_compile_options(lanelet2_traffic_rules PRIVATE ${LANELET2_SUPPRESS_WARNINGS})

# =============================================================================
# lanelet2_routing
# =============================================================================

file(GLOB LANELET2_ROUTING_SRC "${LANELET2_DIR}/lanelet2_routing/src/*.cpp")
add_library(lanelet2_routing ${LANELET2_LIB_TYPE} ${LANELET2_ROUTING_SRC})
target_include_directories(lanelet2_routing PUBLIC
        $<BUILD_INTERFACE:${LANELET2_DIR}/lanelet2_routing/include>
        $<INSTALL_INTERFACE:include>
)
target_link_libraries(lanelet2_routing PUBLIC lanelet2_core lanelet2_traffic_rules Boost::boost)
target_compile_options(lanelet2_routing PRIVATE ${LANELET2_SUPPRESS_WARNINGS})

# =============================================================================
# Namespaced aliases (so consumers can use lanelet2_core::lanelet2_core etc.)
# =============================================================================

add_library(lanelet2_core::lanelet2_core ALIAS lanelet2_core)
add_library(lanelet2_io::lanelet2_io ALIAS lanelet2_io)
add_library(lanelet2_projection::lanelet2_projection ALIAS lanelet2_projection)
add_library(lanelet2_traffic_rules::lanelet2_traffic_rules ALIAS lanelet2_traffic_rules)
add_library(lanelet2_routing::lanelet2_routing ALIAS lanelet2_routing)

# =============================================================================
# Install rules (so consumer export sets can reference lanelet2 targets)
# =============================================================================

include(GNUInstallDirs)

install(TARGETS lanelet2_core lanelet2_io lanelet2_projection lanelet2_traffic_rules lanelet2_routing
        EXPORT lanelet2Targets
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

# Install public headers
install(DIRECTORY ${LANELET2_DIR}/lanelet2_core/include/       DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
install(DIRECTORY ${LANELET2_DIR}/lanelet2_io/include/         DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
install(DIRECTORY ${LANELET2_DIR}/lanelet2_projection/include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
install(DIRECTORY ${LANELET2_DIR}/lanelet2_traffic_rules/include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
install(DIRECTORY ${LANELET2_DIR}/lanelet2_routing/include/    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

install(EXPORT lanelet2Targets
        NAMESPACE lanelet2_
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/lanelet2
)

message(STATUS "lanelet2: Built from source (NO ROS) — core, io, projection, traffic_rules, routing")
