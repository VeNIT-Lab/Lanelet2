cmake_minimum_required(VERSION 3.14)
project(lanelet2 LANGUAGES CXX)

include(GNUInstallDirs)
install(FILES package.xml DESTINATION ${CMAKE_INSTALL_DATADIR}/lanelet2)
