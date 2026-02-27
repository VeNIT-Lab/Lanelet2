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

if(NOT TARGET Boost::boost)
    find_package(Boost REQUIRED)
endif()

if(NOT TARGET Eigen3::Eigen)
    find_package(Eigen3 REQUIRED)
endif()

file(GLOB LANELET2_CORE_SOURCES CONFIGURE_DEPENDS src/*.cpp)

add_library(lanelet2_core ${LANELET2_LIB_TYPE} ${LANELET2_CORE_SOURCES})
add_library(lanelet2_core::lanelet2_core ALIAS lanelet2_core)

target_include_directories(
        lanelet2_core
        PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>)

target_link_libraries(lanelet2_core PUBLIC Boost::boost Eigen3::Eigen)
target_compile_features(lanelet2_core PUBLIC cxx_std_14)

if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    target_compile_options(lanelet2_core PRIVATE ${LANELET2_SUPPRESS_WARNINGS})
endif()

set_target_properties(lanelet2_core PROPERTIES SOVERSION 1 VERSION 1.3.0)

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

set(_lanelet2_core_cmake_dir "${CMAKE_INSTALL_LIBDIR}/cmake/lanelet2_core")
set(_lanelet2_core_config_in "${CMAKE_CURRENT_SOURCE_DIR}/cmake/lanelet2_coreConfig.cmake.in")
set(_lanelet2_core_config "${CMAKE_CURRENT_BINARY_DIR}/lanelet2_coreConfig.cmake")
set(_lanelet2_core_config_version "${CMAKE_CURRENT_BINARY_DIR}/lanelet2_coreConfigVersion.cmake")

configure_package_config_file(
        "${_lanelet2_core_config_in}"
        "${_lanelet2_core_config}"
        INSTALL_DESTINATION "${_lanelet2_core_cmake_dir}")

write_basic_package_version_file(
        "${_lanelet2_core_config_version}"
        VERSION 1.3.0
        COMPATIBILITY SameMajorVersion)

install(
        TARGETS lanelet2_core
        EXPORT lanelet2_coreTargets
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

install(
        EXPORT lanelet2_coreTargets
        FILE lanelet2_coreTargets.cmake
        DESTINATION "${_lanelet2_core_cmake_dir}")

install(
        FILES "${_lanelet2_core_config}" "${_lanelet2_core_config_version}"
        DESTINATION "${_lanelet2_core_cmake_dir}")

install(
        DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
