cmake_minimum_required(VERSION 3.14.0 FATAL_ERROR)
project(sb6code DESCRIPTION "OpenGL SuperBible 6th Edition Examples" VERSION 6.0.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

if(WIN32)
  add_definitions(-D_UNICODE -DUNICODE -DWIN32_LEAN_AND_MEAN -DNOMINMAX -DWINVER=0x0A00 -D_WIN32_WINNT=0x0A00)
  add_definitions(-D_CRT_SECURE_NO_DEPRECATE -D_CRT_SECURE_NO_WARNINGS -D_CRT_NONSTDC_NO_DEPRECATE)
  add_definitions(-D_ATL_SECURE_NO_DEPRECATE -D_SCL_SECURE_NO_WARNINGS)
endif()

set_property(GLOBAL PROPERTY USE_FOLDERS ON)
set_property(GLOBAL PROPERTY PREDEFINED_TARGETS_FOLDER build)
set_property(DIRECTORY ${CMAKE_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT starfield)

file(GLOB headers CONFIGURE_DEPENDS include/*.h)
source_group(TREE ${CMAKE_CURRENT_SOURCE_DIR}/include PREFIX "include" FILES ${headers})

file(GLOB sources CONFIGURE_DEPENDS src/sb6/*.h src/sb6/*.cpp)
source_group(TREE ${CMAKE_CURRENT_SOURCE_DIR}/src/sb6 PREFIX "src" FILES ${sources})

add_library(sb6 ${headers} ${sources})
target_include_directories(sb6 PUBLIC include)

if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
  find_package(X11 REQUIRED)
  target_link_libraries(sb6 PUBLIC X11::X11 dl)
endif()

find_package(unofficial-angle CONFIG REQUIRED)
target_link_libraries(sb6 PUBLIC unofficial::angle::libEGL unofficial::angle::libGLESv2)

find_package(glfw3 CONFIG REQUIRED)
target_link_libraries(sb6 PUBLIC glfw)

# TODO: Figure out if 'GLFW_DLL' should be define in case of 'VCPKG_LIBRARY_LINKAGE dynamic'.

set(examples
  alienrain
  basicfbo
  blendmatrix
  blinnphong
  bumpmapping
  clipdistance
  csflocking
  cubemapenv
  cubicbezier
  deferredshading
  depthclamp
  dispmap
  dof
  envmapsphere
  equirectangular
  fragcolorfrompos
  fragmentlist
  grass
  gsculling
  gslayered
  gsquads
  gstessellate
  hdrbloom
  hdrexposure
  hdrtonemap
  indexedcube
  instancedattribs
  julia
  ktxview
  linesmooth
  movingtri
  msaanative
  multidrawindirect
  multimaterial
  multiscissor
  multiviewport
  noise
  noperspective
  normalviewer
  objectexploder
  perf-readpixels
  perpixelgloss
  phonglighting
  polygonsmooth
  prefixsum
  prefixsum2d
  programinfo
  raytracer
  rimlight
  sampleshading
# sb6mrender
  shadowmapping
  shapedpoints
  simpleclear
  simpletexcoords
  simpletexture
  singlepoint
  singletri
  sparsetexture
  spinnycube
  springmass
  ssao
  starfield
  stereo
  subroutines
  tessellatedcube
  tessellatedgstri
  tessellatedtri
  tessmodes
  tesssubdivmodes
  toonshading
  tunnel
  wrapmodes
)

if (WIN32)
  list(APPEND examples win32raw)
endif()

foreach(example ${examples})
  file(GLOB example_sources CONFIGURE_DEPENDS src/${example}/*.cpp)
  source_group(TREE ${CMAKE_CURRENT_SOURCE_DIR}/src/${example} PREFIX "" FILES ${example_sources})
  add_executable(${example} WIN32 ${example_sources})
  set_target_properties(${example} PROPERTIES VS_DEBUGGER_WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}/bin)
  target_link_libraries(${example} PRIVATE sb6)
  install(TARGETS ${example} CONFIGURATIONS Release RUNTIME DESTINATION bin)
endforeach()

if(WIN32)
  install(CODE [[
    file(GLOB libraries ${CMAKE_BINARY_DIR}/*.dll ${CMAKE_BINARY_DIR}/Release/*.dll)
    file(INSTALL ${libraries} DESTINATION ${CMAKE_INSTALL_PREFIX}/bin)
  ]])
endif()
