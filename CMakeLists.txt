cmake_minimum_required(VERSION 2.8 FATAL_ERROR)

project(cadmesh)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 ")

set(CADMESH_MAJOR_VERSION 2)
set(CADMESH_MINOR_VERSION 0)
set(CADMESH_PATCH_VERSION 0)
set(CADMESH_VERSION ${CADMESH_MAJOR_VERSION}.${CADMESH_MINOR_VERSION}.${CADMESH_PATCH_VERSION})

include(FindPkgConfig)
include(ExternalProject)

set(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake/Modules ${CMAKE_MODULE_PATH})

# Make the include and lib directories in the build area.
file(MAKE_DIRECTORY ${CMAKE_BINARY_DIR}/include)
file(MAKE_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

# GEANT4
option(WITH_GEANT4_UIVIS "Build with Geant4 UI and VIS drivers" ON)
if(WITH_GEANT4_UIVIS)
    find_package(Geant4 REQUIRED ui_all vis_all)
else()
    find_package(Geant4 REQUIRED)
endif()
include(${Geant4_USE_FILE})

# X11 if on a Mac
if(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
    set(OPENGL_INCLUDE_DIR_MAC /var/empty/X11/include )
    include_directories(${OPENGL_INCLUDE_DIR_MAC})

    set(OPENGL_INCLUDE_DIR /var/empty/include )
    include_directories(${OPENGL_INCLUDE_DIR})
endif()

# Tetgen
option(WITH_SYS_TETGEN "WITH_TETGEN" OFF)
if(${WITH_SYS_TETGEN} MATCHES "ON")
    find_package(tetgen)
endif()

# Open Asset Importer Library
option(WITH_SYS_ASSIMP "WITH_ASSIMP" OFF)
if(${WITH_SYS_ASSIMP} MATCHES "ON")
    find_package(assimp)
endif()

# Catch unit testing library
option(WITH_TESTS "WITH_TESTS" OFF)
if(${WITH_TESTS} MATCHES "ON")
    ExternalProject_Add(
        catch_external
        SOURCE_DIR ${PROJECT_SOURCE_DIR}/external/catch
        GIT_REPOSITORY https://github.com/catchorg/Catch2.git
        GIT_TAG v2.11.3
        CMAKE_ARGS -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_BINARY_DIR} -DNO_SELFTEST=ON
    )
    include_directories(${PROJECT_BINARY_DIR}/include/catch)
endif()

# CADMesh
set(DEFAULT_READER "BuiltIn" CACHE STRING "The CAD file reader to use by default.")
configure_file(${PROJECT_SOURCE_DIR}/include/Configuration.hh.in "${CMAKE_CURRENT_BINARY_DIR}/include/Configuration.hh" @ONLY)

file(GLOB sources ${PROJECT_SOURCE_DIR}/src/*.cc)
file(GLOB headers ${PROJECT_SOURCE_DIR}/include/*.hh)

include_directories(${PROJECT_SOURCE_DIR}/include)
include_directories(${CMAKE_INSTALL_PREFIX}/include)
include_directories(${PROJECT_BINARY_DIR}/include)

# Set up the target.
add_library(cadmesh "SHARED" ${sources} ${headers}) 

# Link against either the system libraries, or the external project libaries.
if(${WITH_SYS_TETGEN} MATCHES "ON")
    target_link_libraries(cadmesh tet)
endif()

if(${WITH_SYS_ASSIMP} MATCHES "ON")
    target_link_libraries(cadmesh assimp)
endif()

target_link_libraries(cadmesh ${Geant4_LIBRARIES})

# Build the examples as well.
if(${WITH_EXAMPLES} MATCHES "ON")
    add_subdirectory(${PROJECT_SOURCE_DIR}/examples/basic)
endif()

# Configure the installation.
configure_file("${CMAKE_CURRENT_SOURCE_DIR}/cmake/cadmesh-config.cmake.in" "${CMAKE_CURRENT_BINARY_DIR}/cadmesh-config.cmake" @ONLY IMMEDIATE)
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/cadmesh-config.cmake" DESTINATION "lib/cmake/cadmesh-${CADMESH_MAJOR_VERSION}.${CADMESH_MINOR_VERSION}.${CADMESH_PATCH_VERSION}")

# Install the CADMesh headers and library.
install(FILES ${headers} DESTINATION include/CADMesh)
install(FILES ${PROJECT_BINARY_DIR}/include/Configuration.hh DESTINATION include/CADMesh)
install(TARGETS cadmesh DESTINATION lib)

# Set CPack to build an archive.
set(CPACK_GENERATOR "TGZ")
set(CPACK_PACKAGE_NAME "cadmesh")
set(CPACK_PACKAGE_VERSION ${CADMESH_VERSION})
set(CPACK_PACKAGE_VERSION_MAJOR ${CADMESH_MAJOR_VERSION})
set(CPACK_PACKAGE_VERSION_MINOR ${CADMESH_MINOR_VERSION})
set(CPACK_PACKAGE_VERSION_PATCH ${CADMESH_PATCH_VERSION})
set(CPACK_INCLUDE_TOPLEVEL_DIRECTORY 0)
include(CPack)

# Tests
if(${WITH_TESTS} MATCHES "ON")
    enable_testing()

    add_executable(CADMeshTests tests/CADMeshTests.cc)
    add_dependencies(CADMeshTests catch_external)
    target_link_libraries(CADMeshTests cadmesh)
    add_test(NAME CADMeshTests COMMAND CADMeshTests)

    add_executable(ValidTessellatedMeshTests tests/ValidTessellatedMeshTests.cc)
    add_dependencies(ValidTessellatedMeshTests catch_external)
    target_link_libraries(ValidTessellatedMeshTests cadmesh)
    add_test(NAME ValidTessellatedMeshTests COMMAND ValidTessellatedMeshTests)

    add_executable(InvalidTessellatedMeshTests tests/InvalidTessellatedMeshTests.cc)
    add_dependencies(InvalidTessellatedMeshTests catch_external)
    target_link_libraries(InvalidTessellatedMeshTests cadmesh)
    add_test(NAME InvalidTessellatedMeshTests COMMAND InvalidTessellatedMeshTests)

endif()

