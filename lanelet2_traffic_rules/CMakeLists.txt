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

include(GNUInstallDirs)
install(
        TARGETS lanelet2_traffic_rules
        EXPORT ${LANELET2_EXPORT_SET}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
