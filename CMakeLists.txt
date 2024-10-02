cmake_minimum_required (VERSION 3.15)
project (Flow VERSION 0.1 LANGUAGES CXX)

# enable C++11
set(CMAKE_CXX_STANDARD 14)
add_definitions(-D_CRT_SECURE_NO_WARNINGS)

# 导出compile_commands.json文件
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

set_directory_properties(PROPERTIES COMPILE_DEFINITIONS_DEBUG "_DEBUG")

find_package(glfw3 REQUIRED)
find_package(Vulkan REQUIRED)

# include flow header and source files
include_directories("${PROJECT_SOURCE_DIR}/flow")
file(GLOB SOURCES "flow/*.cpp")

# create the executable
add_executable(Flow ${SOURCES})

# add libraries (note that as of version 3.7, CMake supports Vulkan out-of-the-box)
target_link_libraries(Flow glfw Vulkan::Vulkan)

# add a pre-build step to compile shaders
set(SHADER_DIR ${CMAKE_CURRENT_SOURCE_DIR}/flow/shaders)
set(SHADER_OUTPUT_DIR ${CMAKE_CURRENT_BINARY_DIR}/flow/shaders)
file(GLOB_RECURSE SHADER_FILES ${SHADER_DIR}/*.frag ${SHADER_DIR}/*.vert)

foreach(SHADER_FILE ${SHADER_FILES})
    get_filename_component(FILE_NAME ${SHADER_FILE} NAME_WE)
    set(SHADER_OUTPUT_FILE ${SHADER_OUTPUT_DIR}/${FILE_NAME}.spv)
    file(MAKE_DIRECTORY ${SHADER_OUTPUT_DIR})
    add_custom_command(
        OUTPUT ${SHADER_OUTPUT_FILE}
        DEPENDS ${SHADER_FILE}
        COMMAND glslangValidator -V -o ${SHADER_OUTPUT_FILE} ${SHADER_FILE}  
    )
    message(STATUS "${SHADER_OUTPUT_FILE}")
    list(APPEND SHADER_OUTPUT_FILES ${SHADER_OUTPUT_FILE})
endforeach()

add_custom_target(Shaders DEPENDS ${SHADER_OUTPUT_FILES})
add_dependencies(Flow Shaders)
