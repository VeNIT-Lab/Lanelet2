if(NOT DEFINED LANELET2_EXPORT_SET)
    set(LANELET2_EXPORT_SET lanelet2Targets)
endif()

add_library(lanelet2_maps INTERFACE)
add_library(lanelet2_maps::lanelet2_maps ALIAS lanelet2_maps)

include(GNUInstallDirs)
install(TARGETS lanelet2_maps EXPORT ${LANELET2_EXPORT_SET})
install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/res/ DESTINATION ${CMAKE_INSTALL_DATADIR}/lanelet2_maps/res)
install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/josm/ DESTINATION ${CMAKE_INSTALL_DATADIR}/lanelet2_maps/josm)
