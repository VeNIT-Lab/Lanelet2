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

include(GNUInstallDirs)
install(
        TARGETS lanelet2_projection
        EXPORT ${LANELET2_EXPORT_SET}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
