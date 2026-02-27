if(NOT DEFINED LANELET2_LIB_TYPE)
    option(LANELET2_SHARED_LIBS "Build lanelet2 as shared libraries (.so)" ON)
    if(LANELET2_SHARED_LIBS)
        set(LANELET2_LIB_TYPE SHARED)
    else()
        set(LANELET2_LIB_TYPE STATIC)
    endif()
endif()

if(NOT DEFINED LANELET2_SUPPRESS_WARNINGS)
    set(LANELET2_SUPPRESS_WARNINGS
            -Wno-deprecated-declarations
            -Wno-unused-parameter
            -Wno-sign-compare)
endif()

if(NOT TARGET lanelet2_core OR NOT TARGET lanelet2_traffic_rules)
    message(FATAL_ERROR "lanelet2_routing requires lanelet2_core and lanelet2_traffic_rules.")
endif()

if(NOT TARGET Boost::boost)
    find_package(Boost REQUIRED)
endif()

file(GLOB LANELET2_ROUTING_SOURCES CONFIGURE_DEPENDS src/*.cpp)

add_library(lanelet2_routing ${LANELET2_LIB_TYPE} ${LANELET2_ROUTING_SOURCES})
add_library(lanelet2_routing::lanelet2_routing ALIAS lanelet2_routing)

target_include_directories(
        lanelet2_routing
        PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>)

target_link_libraries(lanelet2_routing PUBLIC lanelet2_core lanelet2_traffic_rules Boost::boost)
target_compile_features(lanelet2_routing PUBLIC cxx_std_14)

if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    target_compile_options(lanelet2_routing PRIVATE ${LANELET2_SUPPRESS_WARNINGS})
endif()

set_target_properties(lanelet2_routing PROPERTIES SOVERSION 1 VERSION 1.3.0)
include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

set(_lanelet2_routing_cmake_dir "${CMAKE_INSTALL_LIBDIR}/cmake/lanelet2_routing")
set(_lanelet2_routing_config_in "${CMAKE_CURRENT_SOURCE_DIR}/cmake/lanelet2_routingConfig.cmake.in")
set(_lanelet2_routing_config "${CMAKE_CURRENT_BINARY_DIR}/lanelet2_routingConfig.cmake")
set(_lanelet2_routing_config_version "${CMAKE_CURRENT_BINARY_DIR}/lanelet2_routingConfigVersion.cmake")

configure_package_config_file(
        "${_lanelet2_routing_config_in}"
        "${_lanelet2_routing_config}"
        INSTALL_DESTINATION "${_lanelet2_routing_cmake_dir}")

write_basic_package_version_file(
        "${_lanelet2_routing_config_version}"
        VERSION 1.3.0
        COMPATIBILITY SameMajorVersion)

install(
        TARGETS lanelet2_routing
        EXPORT lanelet2_routingTargets
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

install(
        EXPORT lanelet2_routingTargets
        FILE lanelet2_routingTargets.cmake
        DESTINATION "${_lanelet2_routing_cmake_dir}")

install(
        FILES "${_lanelet2_routing_config}" "${_lanelet2_routing_config_version}"
        DESTINATION "${_lanelet2_routing_cmake_dir}")

install(
        DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
