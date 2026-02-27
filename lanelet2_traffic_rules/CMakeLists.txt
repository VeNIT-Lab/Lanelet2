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

if(NOT TARGET lanelet2_core)
    message(FATAL_ERROR "lanelet2_traffic_rules requires lanelet2_core.")
endif()

file(GLOB LANELET2_TRAFFIC_RULES_SOURCES CONFIGURE_DEPENDS src/*.cpp)

add_library(lanelet2_traffic_rules ${LANELET2_LIB_TYPE} ${LANELET2_TRAFFIC_RULES_SOURCES})
add_library(lanelet2_traffic_rules::lanelet2_traffic_rules ALIAS lanelet2_traffic_rules)

target_include_directories(
        lanelet2_traffic_rules
        PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>)

target_link_libraries(lanelet2_traffic_rules PUBLIC lanelet2_core)
target_compile_features(lanelet2_traffic_rules PUBLIC cxx_std_14)

if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    target_compile_options(lanelet2_traffic_rules PRIVATE ${LANELET2_SUPPRESS_WARNINGS})
endif()

set_target_properties(lanelet2_traffic_rules PROPERTIES SOVERSION 1 VERSION 1.3.0)
include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

set(_lanelet2_traffic_rules_cmake_dir "${CMAKE_INSTALL_LIBDIR}/cmake/lanelet2_traffic_rules")
set(_lanelet2_traffic_rules_config_in "${CMAKE_CURRENT_SOURCE_DIR}/cmake/lanelet2_traffic_rulesConfig.cmake.in")
set(_lanelet2_traffic_rules_config "${CMAKE_CURRENT_BINARY_DIR}/lanelet2_traffic_rulesConfig.cmake")
set(_lanelet2_traffic_rules_config_version
    "${CMAKE_CURRENT_BINARY_DIR}/lanelet2_traffic_rulesConfigVersion.cmake")

configure_package_config_file(
        "${_lanelet2_traffic_rules_config_in}"
        "${_lanelet2_traffic_rules_config}"
        INSTALL_DESTINATION "${_lanelet2_traffic_rules_cmake_dir}")

write_basic_package_version_file(
        "${_lanelet2_traffic_rules_config_version}"
        VERSION 1.3.0
        COMPATIBILITY SameMajorVersion)

install(
        TARGETS lanelet2_traffic_rules
        EXPORT lanelet2_traffic_rulesTargets
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

install(
        EXPORT lanelet2_traffic_rulesTargets
        FILE lanelet2_traffic_rulesTargets.cmake
        DESTINATION "${_lanelet2_traffic_rules_cmake_dir}")

install(
        FILES "${_lanelet2_traffic_rules_config}" "${_lanelet2_traffic_rules_config_version}"
        DESTINATION "${_lanelet2_traffic_rules_cmake_dir}")

install(
        DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
