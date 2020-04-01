cmake_minimum_required (VERSION 2.8)
cmake_policy(VERSION 2.8)

project (PlayGL)

IF(NOT CMAKE_BUILD_TYPE)
  SET(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build (Debug or Release)" FORCE)
ENDIF(NOT CMAKE_BUILD_TYPE)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/modules/")

if(WIN32)
	set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/bin)
endif(WIN32)

link_directories(${CMAKE_SOURCE_DIR}/lib)

list(APPEND CMAKE_CXX_FLAGS "-std=c++11")

# find the required packages
find_package(GLM REQUIRED)
message(STATUS "GLM included at ${GLM_INCLUDE_DIR}")
find_package(GLFW3 REQUIRED)
message(STATUS "Found GLFW3 in ${GLFW3_INCLUDE_DIR}")
find_package(ASSIMP REQUIRED)
message(STATUS "Found ASSIMP in ${ASSIMP_INCLUDE_DIR}")

set(LIBS glfw3 opengl32 assimp)

set(PlayGlDemo
    simple
)

configure_file(configuration/root_directory.h.in configuration/root_directory.h)
include_directories(${CMAKE_BINARY_DIR}/configuration)

# first create relevant static libraries requried for other projects
add_library(STB_IMAGE "src/stb_image.cpp")
set(LIBS ${LIBS} STB_IMAGE)

add_library(GLAD "src/glad.c")
set(LIBS ${LIBS} GLAD)

macro(makeLink src dest target)
  add_custom_command(TARGET ${target} POST_BUILD COMMAND ${CMAKE_COMMAND} -E create_symlink ${src} ${dest}  DEPENDS  ${dest} COMMENT "mklink ${src} -> ${dest}")
endmacro()

# create a project file
foreach(DEMO ${PlayGlDemo})
    file(GLOB SOURCE
        "src/playgl/${DEMO}/*.h"
        "src/playgl/${DEMO}/*.cc"
        "src/playgl/${DEMO}/*.cpp"
        "src/playgl/${DEMO}/*.vs"
        "src/playgl/${DEMO}/*.fs"
        "src/playgl/${DEMO}/*.gs"
    )
    set(NAME "playgl__${DEMO}")
    add_executable(${NAME} ${SOURCE})
    target_link_libraries(${NAME} ${LIBS})
    set_target_properties(${NAME} PROPERTIES RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/bin/playgl")

    # copy shader files to build directory
    file(GLOB SHADERS
                "src/playgl/${DEMO}/*.vs"
                # "src/playgl/${DEMO}/*.frag"
                "src/playgl/${DEMO}/*.fs"
                "src/playgl/${DEMO}/*.gs"
    )
    foreach(SHADER ${SHADERS})
        add_custom_command(TARGET ${NAME} PRE_BUILD COMMAND ${CMAKE_COMMAND} -E copy ${SHADER} $<TARGET_FILE_DIR:${NAME}>)
    endforeach(SHADER)
    # if compiling for visual studio, also use configure file for each project (specifically to set up working directory)
    configure_file(${CMAKE_SOURCE_DIR}/configuration/visualstudio.vcxproj.user.in ${CMAKE_CURRENT_BINARY_DIR}/${NAME}.vcxproj.user @ONLY)
endforeach(DEMO)


include_directories(${CMAKE_SOURCE_DIR}/includes)
