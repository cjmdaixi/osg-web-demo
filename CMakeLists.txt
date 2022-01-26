CMAKE_MINIMUM_REQUIRED(VERSION 2.8)

# Path to OpenSceneGraph
SET(EXT_PROJ_DIR "${CMAKE_SOURCE_DIR}/../..")

# Specify critical OpenSceneGraph build variables.
SET(BUILD_OSG_APPLICATIONS NO CACHE BOOL "Do not build applications")
SET(EGL_LIBRARY "GL" CACHE STRING "Suppress linkage error")
SET(OSG_GL1_AVAILABLE OFF CACHE BOOL "Unavailable under Emscripten")
SET(OSG_GL2_AVAILABLE OFF CACHE BOOL "Unavailable under Emscripten")
SET(OSG_GLES2_AVAILABLE ON CACHE BOOL "GLES2 is what Emscripten uses")
SET(OSG_WINDOWING_SYSTEM "None" CACHE STRING "Since we use SDL2 for OpenGL context creation, we don't have a windowing system")
SET(DYNAMIC_OPENTHREADS OFF CACHE BOOL "Link OpenThreads statically")
SET(DYNAMIC_OPENSCENEGRAPH OFF CACHE BOOL "Link OpenSceneGraph statically")
# Prevent CMake error during configuration.
SET(_OPENTHREADS_ATOMIC_USE_GCC_BUILTINS_EXITCODE "0" CACHE STRING "Prevent cfg error")

# Reference SDL2 during build process.
# We use SDL2 to do the following:
# * OpenGL functions' address retrieval
# * OpenGL graphics context creation
SET(WASM_FLAGS "-s USE_SDL=2 -s ERROR_ON_UNDEFINED_SYMBOLS=0")
SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${WASM_FLAGS}" CACHE STRING "Use SDL2" FORCE)

# Build and reference OpenSceneGraph.
find_package(OpenSceneGraph REQUIRED osgDB osgUtil osgViewer osgGA)
message(STATUS "OPENSCENEGRAPH_INCLUDE_DIRS=${OPENSCENEGRAPH_INCLUDE_DIRS}")
message(STATUS "OPENSCENEGRAPH_LIBRARIES=${OPENSCENEGRAPH_LIBRARIES}")

# Build sample.
INCLUDE_DIRECTORIES(src-gen)
ADD_EXECUTABLE(sample-ems src/main.cpp)
# Make Emscripten generate ready-to-open HTML page.
SET(CMAKE_EXECUTABLE_SUFFIX ".html")
# Make Emscripten preload the resource.
SET_TARGET_PROPERTIES(sample-ems PROPERTIES LINK_FLAGS "--preload-file box.osgt")
# Copy the resource to the build directory.
CONFIGURE_FILE(box.osgt box.osgt COPYONLY)
# Libraries must be linked in the specified order.
# Otherwise you may get unsatisified linker errors.
target_include_directories(sample-ems PRIVATE ${OPENSCENEGRAPH_INCLUDE_DIRS})

TARGET_LINK_LIBRARIES(
    sample-ems
    ${OPENSCENEGRAPH_LIBRARIES}
    C:/Users/Administrator/Desktop/WASM/emsdk/upstream/emscripten/cache/sysroot/lib/osgPlugins-3.6.5/libosgdb_osg.a
    C:/Users/Administrator/Desktop/WASM/emsdk/upstream/emscripten/cache/sysroot/lib/osgPlugins-3.6.5/libosgdb_serializers_osg.a
)
