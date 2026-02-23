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

include(GNUInstallDirs)
install(
        TARGETS lanelet2_routing
        EXPORT ${LANELET2_EXPORT_SET}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
