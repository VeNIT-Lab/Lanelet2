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

if(NOT TARGET lanelet2_core OR NOT TARGET lanelet2_io OR NOT TARGET lanelet2_projection OR NOT TARGET lanelet2_routing
        OR NOT TARGET lanelet2_traffic_rules)
    message(
            FATAL_ERROR
            "lanelet2_validation requires lanelet2_core, lanelet2_io, lanelet2_projection, lanelet2_routing and lanelet2_traffic_rules.")
endif()

if(NOT TARGET Boost::program_options)
    find_package(Boost REQUIRED COMPONENTS program_options)
endif()

file(GLOB_RECURSE LANELET2_VALIDATION_SOURCES CONFIGURE_DEPENDS src/*.cpp)

add_library(lanelet2_validation ${LANELET2_LIB_TYPE} ${LANELET2_VALIDATION_SOURCES})
add_library(lanelet2_validation::lanelet2_validation ALIAS lanelet2_validation)

target_include_directories(
        lanelet2_validation
        PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>)

target_link_libraries(
        lanelet2_validation
        PUBLIC lanelet2_core
        lanelet2_io
        lanelet2_projection
        lanelet2_routing
        lanelet2_traffic_rules
        Boost::program_options)
target_compile_features(lanelet2_validation PUBLIC cxx_std_14)

if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    target_compile_options(lanelet2_validation PRIVATE ${LANELET2_SUPPRESS_WARNINGS})
endif()

add_executable(lanelet2_validate tools/lanelet2_validate/main.cpp)
target_link_libraries(lanelet2_validate PRIVATE lanelet2_validation)
target_compile_features(lanelet2_validate PRIVATE cxx_std_14)

if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    target_compile_options(lanelet2_validate PRIVATE ${LANELET2_SUPPRESS_WARNINGS})
endif()

set_target_properties(lanelet2_validation PROPERTIES SOVERSION 1 VERSION 1.3.0)
include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

set(_lanelet2_validation_cmake_dir "${CMAKE_INSTALL_LIBDIR}/cmake/lanelet2_validation")
set(_lanelet2_validation_config_in "${CMAKE_CURRENT_SOURCE_DIR}/cmake/lanelet2_validationConfig.cmake.in")
set(_lanelet2_validation_config "${CMAKE_CURRENT_BINARY_DIR}/lanelet2_validationConfig.cmake")
set(_lanelet2_validation_config_version
    "${CMAKE_CURRENT_BINARY_DIR}/lanelet2_validationConfigVersion.cmake")

configure_package_config_file(
        "${_lanelet2_validation_config_in}"
        "${_lanelet2_validation_config}"
        INSTALL_DESTINATION "${_lanelet2_validation_cmake_dir}")

write_basic_package_version_file(
        "${_lanelet2_validation_config_version}"
        VERSION 1.3.0
        COMPATIBILITY SameMajorVersion)

install(
        TARGETS lanelet2_validation
        EXPORT lanelet2_validationTargets
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

install(
        EXPORT lanelet2_validationTargets
        FILE lanelet2_validationTargets.cmake
        DESTINATION "${_lanelet2_validation_cmake_dir}")

install(
        FILES "${_lanelet2_validation_config}" "${_lanelet2_validation_config_version}"
        DESTINATION "${_lanelet2_validation_cmake_dir}")

install(TARGETS lanelet2_validate RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

install(
        DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
