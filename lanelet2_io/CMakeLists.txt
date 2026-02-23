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
    message(FATAL_ERROR "lanelet2_io requires lanelet2_core. Add lanelet2_core first.")
endif()

if(NOT TARGET Boost::filesystem OR NOT TARGET Boost::serialization)
    find_package(Boost REQUIRED COMPONENTS filesystem serialization)
endif()

if(NOT TARGET pugixml::pugixml)
    find_package(pugixml REQUIRED)
endif()

file(GLOB LANELET2_IO_SOURCES CONFIGURE_DEPENDS src/*.cpp)

add_library(lanelet2_io ${LANELET2_LIB_TYPE} ${LANELET2_IO_SOURCES})
add_library(lanelet2_io::lanelet2_io ALIAS lanelet2_io)

target_include_directories(
        lanelet2_io
        PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>)

set(_lanelet2_pugixml_target pugixml::pugixml)
if(NOT TARGET pugixml::pugixml)
    if(TARGET pugixml)
        set(_lanelet2_pugixml_target pugixml)
    else()
        set(_lanelet2_pugixml_target pugixml)
    endif()
endif()

target_link_libraries(
        lanelet2_io
        PUBLIC lanelet2_core Boost::filesystem Boost::serialization
        PRIVATE ${_lanelet2_pugixml_target})
target_compile_features(lanelet2_io PUBLIC cxx_std_14)

if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    target_compile_options(lanelet2_io PRIVATE ${LANELET2_SUPPRESS_WARNINGS})
endif()

include(GNUInstallDirs)
install(
        TARGETS lanelet2_io
        EXPORT ${LANELET2_EXPORT_SET}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
