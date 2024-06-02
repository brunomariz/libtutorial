# Define minimum cmake version
cmake_minimum_required(VERSION 3.22)
# Define a project
project(CmakeExample VERSION 1.0 DESCRIPTION "tutorial library")
# Declare a library
add_library(tutorial SHARED src/libtutorial.c)
# Set the version property (optional)
set_target_properties(tutorial PROPERTIES VERSION ${PROJECT_VERSION})
# Declare the public API (files in inc/). Private header files should be placed in src/ along with the C files.
set_target_properties(tutorial PROPERTIES PUBLIC_HEADER inc/libtutorial.h)
# Include the top level directory
target_include_directories(tutorial PRIVATE inc)
target_include_directories(tutorial PRIVATE src)
# Create an install rule and declare files to install
include(GNUInstallDirs)
install(TARGETS tutorial
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
