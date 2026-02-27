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
    message(FATAL_ERROR "lanelet2_matching requires lanelet2_core and lanelet2_traffic_rules.")
endif()

file(GLOB LANELET2_MATCHING_SOURCES CONFIGURE_DEPENDS src/*.cpp)

add_library(lanelet2_matching ${LANELET2_LIB_TYPE} ${LANELET2_MATCHING_SOURCES})
add_library(lanelet2_matching::lanelet2_matching ALIAS lanelet2_matching)

target_include_directories(
        lanelet2_matching
        PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>
        PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/include/lanelet2_matching)

target_compile_definitions(lanelet2_matching PRIVATE PKG_DIR="${CMAKE_CURRENT_LIST_DIR}")
target_link_libraries(lanelet2_matching PUBLIC lanelet2_core lanelet2_traffic_rules)
target_compile_features(lanelet2_matching PUBLIC cxx_std_14)

if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    target_compile_options(lanelet2_matching PRIVATE ${LANELET2_SUPPRESS_WARNINGS})
endif()

set_target_properties(lanelet2_matching PROPERTIES SOVERSION 1 VERSION 1.3.0)
include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

set(_lanelet2_matching_cmake_dir "${CMAKE_INSTALL_LIBDIR}/cmake/lanelet2_matching")
set(_lanelet2_matching_config_in "${CMAKE_CURRENT_SOURCE_DIR}/cmake/lanelet2_matchingConfig.cmake.in")
set(_lanelet2_matching_config "${CMAKE_CURRENT_BINARY_DIR}/lanelet2_matchingConfig.cmake")
set(_lanelet2_matching_config_version "${CMAKE_CURRENT_BINARY_DIR}/lanelet2_matchingConfigVersion.cmake")

configure_package_config_file(
        "${_lanelet2_matching_config_in}"
        "${_lanelet2_matching_config}"
        INSTALL_DESTINATION "${_lanelet2_matching_cmake_dir}")

write_basic_package_version_file(
        "${_lanelet2_matching_config_version}"
        VERSION 1.3.0
        COMPATIBILITY SameMajorVersion)

install(
        TARGETS lanelet2_matching
        EXPORT lanelet2_matchingTargets
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

install(
        EXPORT lanelet2_matchingTargets
        FILE lanelet2_matchingTargets.cmake
        DESTINATION "${_lanelet2_matching_cmake_dir}")

install(
        FILES "${_lanelet2_matching_config}" "${_lanelet2_matching_config_version}"
        DESTINATION "${_lanelet2_matching_cmake_dir}")

install(
        DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
