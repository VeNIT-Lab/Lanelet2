add_library(lanelet2_maps INTERFACE)
add_library(lanelet2_maps::lanelet2_maps ALIAS lanelet2_maps)

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

set(_lanelet2_maps_cmake_dir "${CMAKE_INSTALL_LIBDIR}/cmake/lanelet2_maps")
set(_lanelet2_maps_config_in "${CMAKE_CURRENT_SOURCE_DIR}/cmake/lanelet2_mapsConfig.cmake.in")
set(_lanelet2_maps_config "${CMAKE_CURRENT_BINARY_DIR}/lanelet2_mapsConfig.cmake")
set(_lanelet2_maps_config_version "${CMAKE_CURRENT_BINARY_DIR}/lanelet2_mapsConfigVersion.cmake")

configure_package_config_file(
        "${_lanelet2_maps_config_in}"
        "${_lanelet2_maps_config}"
        INSTALL_DESTINATION "${_lanelet2_maps_cmake_dir}")

write_basic_package_version_file(
        "${_lanelet2_maps_config_version}"
        VERSION 1.3.0
        COMPATIBILITY SameMajorVersion)

install(TARGETS lanelet2_maps EXPORT lanelet2_mapsTargets)

install(
        EXPORT lanelet2_mapsTargets
        FILE lanelet2_mapsTargets.cmake
        DESTINATION "${_lanelet2_maps_cmake_dir}")

install(
        FILES "${_lanelet2_maps_config}" "${_lanelet2_maps_config_version}"
        DESTINATION "${_lanelet2_maps_cmake_dir}")

install(
        DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/res/
        DESTINATION ${CMAKE_INSTALL_DATADIR}/lanelet2_maps/res)

install(
        DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/josm/
        DESTINATION ${CMAKE_INSTALL_DATADIR}/lanelet2_maps/josm)
