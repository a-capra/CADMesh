cmake_minimum_required(VERSION 2.6 FATAL_ERROR)
project(cadmesh_example)

# GEANT4 core
find_package(Geant4 REQUIRED ui_all vis_all)
include(${Geant4_USE_FILE})
include_directories(${PROJECT_SOURCE_DIR}/include)

# Include CADMesh in your project.
# find_package(cadmesh)
# include_directories(${CADMESH_INCLUDE_DIRS})

# User code
file(GLOB sources ${PROJECT_SOURCE_DIR}/src/*.cc)
file(GLOB headers ${PROJECT_SOURCE_DIR}/include/*.hh)

#----------------------------------------------------------------------------
# Automatically set the runtime path of your executable to your shared library
#
set(CMAKE_INSTALL_RPATH ${CMAKE_INSTALL_PREFIX}/lib)
set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)


add_executable(cadmesh_example cadmesh_example.cc ${sources} ${headers})
target_link_libraries(cadmesh_example ${Geant4_LIBRARIES})

target_link_libraries(cadmesh_example cadmesh)
# OR target_link_libraries(cadmesh_example ${cadmesh_LIBRARIES})

install(TARGETS cadmesh_example DESTINATION examples)
configure_file( macros/vis.mac
                ${CMAKE_INSTALL_PREFIX}/examples/vis.mac
		COPYONLY )