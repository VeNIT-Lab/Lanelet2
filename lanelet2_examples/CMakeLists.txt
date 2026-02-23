if(NOT DEFINED LANELET2_SUPPRESS_WARNINGS)
    set(LANELET2_SUPPRESS_WARNINGS
            -Wno-deprecated-declarations
            -Wno-unused-parameter
            -Wno-sign-compare)
endif()

if(NOT TARGET lanelet2_core OR NOT TARGET lanelet2_io OR NOT TARGET lanelet2_projection
        OR NOT TARGET lanelet2_traffic_rules OR NOT TARGET lanelet2_routing)
    message(
            FATAL_ERROR
            "lanelet2_examples requires lanelet2_core, lanelet2_io, lanelet2_projection, lanelet2_traffic_rules and lanelet2_routing.")
endif()

file(GLOB LANELET2_EXAMPLE_MAINS CONFIGURE_DEPENDS src/*/main.cpp)

include(GNUInstallDirs)
foreach(_main IN LISTS LANELET2_EXAMPLE_MAINS)
    get_filename_component(_example_dir "${_main}" DIRECTORY)
    get_filename_component(_example_name "${_example_dir}" NAME)

    if(_example_name STREQUAL "07_matching" AND NOT TARGET lanelet2_matching)
        message(STATUS "Skipping lanelet2 example '07_matching' because lanelet2_matching is not built.")
        continue()
    endif()

    set(_target lanelet2_example_${_example_name})
    add_executable(${_target} ${_main})

    target_include_directories(${_target} PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/include)
    target_link_libraries(
            ${_target}
            PRIVATE lanelet2_core lanelet2_io lanelet2_projection lanelet2_traffic_rules lanelet2_routing)

    if(TARGET lanelet2_matching)
        target_link_libraries(${_target} PRIVATE lanelet2_matching)
    endif()

    target_compile_definitions(${_target} PRIVATE PKG_DIR="${CMAKE_CURRENT_SOURCE_DIR}")
    target_compile_features(${_target} PRIVATE cxx_std_14)

    if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
        target_compile_options(${_target} PRIVATE ${LANELET2_SUPPRESS_WARNINGS})
    endif()

    install(TARGETS ${_target} RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})
endforeach()
