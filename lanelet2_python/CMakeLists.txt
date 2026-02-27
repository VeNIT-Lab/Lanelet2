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
            "lanelet2_python requires lanelet2_core, lanelet2_io, lanelet2_projection, lanelet2_traffic_rules and lanelet2_routing.")
endif()

if(NOT TARGET lanelet2_matching)
    message(FATAL_ERROR "lanelet2_python requires lanelet2_matching.")
endif()

find_package(Python3 REQUIRED COMPONENTS Interpreter Development)

if(NOT DEFINED LANELET2_PYTHON_TARGET)
    set(_lanelet2_boost_python_candidates
            python${Python3_VERSION_MAJOR}${Python3_VERSION_MINOR}
            python${Python3_VERSION_MAJOR}
            python3
            python)

    unset(LANELET2_PYTHON_TARGET)
    foreach(_component IN LISTS _lanelet2_boost_python_candidates)
        if(NOT TARGET Boost::${_component})
            find_package(Boost QUIET COMPONENTS ${_component})
        endif()
        if(TARGET Boost::${_component})
            set(LANELET2_PYTHON_TARGET Boost::${_component})
            break()
        endif()
    endforeach()
endif()

if(NOT DEFINED LANELET2_PYTHON_TARGET)
    message(FATAL_ERROR "Could not find a usable Boost.Python target for lanelet2_python.")
endif()

file(GLOB LANELET2_PYTHON_SOURCES CONFIGURE_DEPENDS python_api/*.cpp)

add_library(lanelet2_python MODULE ${LANELET2_PYTHON_SOURCES})
set_target_properties(lanelet2_python PROPERTIES PREFIX "" OUTPUT_NAME lanelet2 SOVERSION 1 VERSION 1.3.0)

target_include_directories(lanelet2_python PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/include)
target_link_libraries(
        lanelet2_python
        PRIVATE lanelet2_core
        lanelet2_io
        lanelet2_projection
        lanelet2_traffic_rules
        lanelet2_routing
        lanelet2_matching
        ${LANELET2_PYTHON_TARGET}
        Python3::Python
        Boost::boost)
target_compile_features(lanelet2_python PRIVATE cxx_std_14)

if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    target_compile_options(lanelet2_python PRIVATE ${LANELET2_SUPPRESS_WARNINGS})
endif()

include(GNUInstallDirs)

if(DEFINED Python3_SITEARCH AND NOT Python3_SITEARCH STREQUAL "")
    set(_lanelet2_python_install_dir ${Python3_SITEARCH})
elseif(DEFINED Python3_SITELIB AND NOT Python3_SITELIB STREQUAL "")
    set(_lanelet2_python_install_dir ${Python3_SITELIB})
else()
    set(_lanelet2_python_install_dir
            ${CMAKE_INSTALL_LIBDIR}/python${Python3_VERSION_MAJOR}.${Python3_VERSION_MINOR}/site-packages)
endif()

install(TARGETS lanelet2_python LIBRARY DESTINATION ${_lanelet2_python_install_dir})
install(
        DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/typings/lanelet2/
        DESTINATION ${_lanelet2_python_install_dir}/lanelet2
        FILES_MATCHING
        PATTERN "*.pyi")
