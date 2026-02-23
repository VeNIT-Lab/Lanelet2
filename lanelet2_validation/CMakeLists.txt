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

include(GNUInstallDirs)
install(
        TARGETS lanelet2_validation
        EXPORT ${LANELET2_EXPORT_SET}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

install(TARGETS lanelet2_validate RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})
install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
