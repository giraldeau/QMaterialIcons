cmake_minimum_required(VERSION 3.1)
project(QMaterialIcons)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

include (GNUInstallDirs)

# Find Qt
find_package(Qt4 4.8 REQUIRED QtGui)

# Instruct CMake to run moc automatically when needed.
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTORCC ON)

set(QMI_VERSION_MAJOR 0)
set(QMI_VERSION_MINOR 1)
set(QMI_VERSION_PATCH 0)
set(QMI_VERSION ${QMI_VERSION_MAJOR}.${QMI_VERSION_MINOR}.${QMIR_VERSION_PATCH})

# Options
option(ENABLE_SAMPLES "Build the samples" ON)

# Credit: https://stackoverflow.com/questions/2368811/how-to-set-warning-level-in-cmake/3818084
if(MSVC)
    # Force to always compile with W4
    if(CMAKE_CXX_FLAGS MATCHES "/W[0-4]")
      string(REGEX REPLACE "/W[0-4]" "/W4" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
    else()
      set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W4")
    endif()
elseif(CMAKE_COMPILER_IS_GNUCC OR CMAKE_COMPILER_IS_GNUCXX)
    # Update if necessary
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wno-long-long -pedantic")
endif()

# Library
add_library(qmaterialicons
    src/qmaterialicons.cpp
    src/font.qrc
)
target_link_libraries(qmaterialicons PUBLIC Qt4::QtGui)
target_include_directories(qmaterialicons PUBLIC
    $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/src>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)
set_target_properties(qmaterialicons PROPERTIES
    VERSION ${QMI_VERSION}
    SOVERSION ${QMI_VERSION_MAJOR}
)
#export(TARGETS qmaterialicons FILE "${CMAKE_BINARY_DIR}/qmaterialicons-targets.cmake")

# Sample
if(ENABLE_SAMPLES)
    add_executable(Sample WIN32
        sample/main.cpp
        sample/mainwindow.cpp
    )
    target_link_libraries(Sample Qt4::QtGui qmaterialicons)
endif()

# Installation
install(TARGETS qmaterialicons
    EXPORT qmaterialicons
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT RuntimeLibraries
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT Development
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT RuntimeLibraries
)
install(FILES src/qmaterialicons.h++ DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}")
install(EXPORT qmaterialicons DESTINATION "${CMAKE_INSTALL_LIBDIR}")
install(FILES qmaterialicons-config.cmake DESTINATION "${CMAKE_INSTALL_LIBDIR}")

# Define variables for the pkg-config file
set(PACKAGE_NAME qmaterialicons)
set(PACKAGE_VERSION ${QMI_VERSION})
configure_file(
    "${CMAKE_SOURCE_DIR}/qmaterialicons.pc.cmakein"
    "${CMAKE_BINARY_DIR}/qmaterialicons.pc"
    @ONLY
)
install(
    FILES "${CMAKE_BINARY_DIR}/qmaterialicons.pc"
    DESTINATION "${CMAKE_INSTALL_LIBDIR}/pkgconfig"
)
