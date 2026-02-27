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

if(NOT TARGET lanelet2_core OR NOT TARGET lanelet2_io)
    message(FATAL_ERROR "lanelet2_projection requires lanelet2_core and lanelet2_io.")
endif()

if(NOT DEFINED LANELET2_GEOGRAPHICLIB_TARGET)
    list(APPEND CMAKE_MODULE_PATH "/usr/share/cmake/geographiclib")
    find_package(GeographicLib REQUIRED)
    if(TARGET GeographicLib::GeographicLib)
        set(LANELET2_GEOGRAPHICLIB_TARGET GeographicLib::GeographicLib)
        unset(LANELET2_GEOGRAPHICLIB_INCLUDE_DIRS)
    else()
        set(LANELET2_GEOGRAPHICLIB_TARGET ${GeographicLib_LIBRARIES})
        set(LANELET2_GEOGRAPHICLIB_INCLUDE_DIRS ${GeographicLib_INCLUDE_DIRS})
    endif()
endif()

file(GLOB LANELET2_PROJECTION_SOURCES CONFIGURE_DEPENDS src/*.cpp)

add_library(lanelet2_projection ${LANELET2_LIB_TYPE} ${LANELET2_PROJECTION_SOURCES})
add_library(lanelet2_projection::lanelet2_projection ALIAS lanelet2_projection)

target_include_directories(
        lanelet2_projection
        PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>)

if(DEFINED LANELET2_GEOGRAPHICLIB_INCLUDE_DIRS AND LANELET2_GEOGRAPHICLIB_INCLUDE_DIRS)
    target_include_directories(lanelet2_projection PRIVATE ${LANELET2_GEOGRAPHICLIB_INCLUDE_DIRS})
endif()

target_link_libraries(
        lanelet2_projection
        PUBLIC lanelet2_core lanelet2_io
        PRIVATE ${LANELET2_GEOGRAPHICLIB_TARGET})
target_compile_features(lanelet2_projection PUBLIC cxx_std_14)

if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    target_compile_options(lanelet2_projection PRIVATE ${LANELET2_SUPPRESS_WARNINGS})
endif()

set_target_properties(lanelet2_projection PROPERTIES SOVERSION 1 VERSION 1.3.0)
include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

set(_lanelet2_projection_cmake_dir "${CMAKE_INSTALL_LIBDIR}/cmake/lanelet2_projection")
set(_lanelet2_projection_config_in "${CMAKE_CURRENT_SOURCE_DIR}/cmake/lanelet2_projectionConfig.cmake.in")
set(_lanelet2_projection_config "${CMAKE_CURRENT_BINARY_DIR}/lanelet2_projectionConfig.cmake")
set(_lanelet2_projection_config_version "${CMAKE_CURRENT_BINARY_DIR}/lanelet2_projectionConfigVersion.cmake")

configure_package_config_file(
        "${_lanelet2_projection_config_in}"
        "${_lanelet2_projection_config}"
        INSTALL_DESTINATION "${_lanelet2_projection_cmake_dir}")

write_basic_package_version_file(
        "${_lanelet2_projection_config_version}"
        VERSION 1.3.0
        COMPATIBILITY SameMajorVersion)

install(
        TARGETS lanelet2_projection
        EXPORT lanelet2_projectionTargets
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

install(
        EXPORT lanelet2_projectionTargets
        FILE lanelet2_projectionTargets.cmake
        DESTINATION "${_lanelet2_projection_cmake_dir}")

install(
        FILES "${_lanelet2_projection_config}" "${_lanelet2_projection_config_version}"
        DESTINATION "${_lanelet2_projection_cmake_dir}")

install(
        DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
