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

if(NOT DEFINED LANELET2_EXPORT_SET)
    set(LANELET2_EXPORT_SET lanelet2Targets)
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

include(GNUInstallDirs)
install(
        TARGETS lanelet2_matching
        EXPORT ${LANELET2_EXPORT_SET}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
