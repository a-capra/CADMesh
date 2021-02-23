cmake_minimum_required(VERSION 2.6 FATAL_ERROR)
project(basic)

set(CMAKE_BUILD_TYPE Debug)

find_package(Geant4 REQUIRED ui_all vis_all)
include(${Geant4_USE_FILE})
include_directories(${CMAKE_SOURCE_DIR}/include)

set(MESHES
  ${CMAKE_SOURCE_DIR}/meshes/sphere.ply ${CMAKE_SOURCE_DIR}/meshes/bunny.stl
  ${CMAKE_SOURCE_DIR}/meshes/cow.obj
  )

#----------------------------------------------------------------------------
# Automatically set the runtime path of your executable to your shared library
#
set(CMAKE_INSTALL_RPATH ${CMAKE_INSTALL_LIBDIR} ${CMAKE_INSTALL_PREFIX}/lib)
set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)
#  set the RPATH before defining the targets - so above


add_executable(basic basic.cc)
target_link_libraries(basic ${Geant4_LIBRARIES} cadmesh)
install(TARGETS basic DESTINATION bin)
install(FILES ${MESHES} DESTINATION bin)


