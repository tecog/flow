cmake_minimum_required (VERSION 2.6)
project (Flow)

# enable C++11
add_definitions(-D_CRT_SECURE_NO_WARNINGS)
add_definitions(-std=c++14)
set_directory_properties(PROPERTIES COMPILE_DEFINITIONS_DEBUG "_DEBUG")

# include flow header and source files
include_directories("${PROJECT_SOURCE_DIR}/flow")
file(GLOB SOURCES "flow/*.cpp")

# setup GLFW
add_subdirectory("${PROJECT_SOURCE_DIR}/third_party/glfw")
include_directories("${PROJECT_SOURCE_DIR}/third_party/glfw/include/GLFW")

# create the executable
add_executable(Flow ${SOURCES})

# add libraries (note that as of version 3.7, CMake supports Vulkan out-of-the-box)
find_package(Vulkan REQUIRED)
target_link_libraries(Flow Vulkan::Vulkan)
target_link_libraries(Flow glfw ${GLFW_LIBRARIES})

#  add a pre-build step to compile shaders
set(SHADER_DIR ${CMAKE_CURRENT_SOURCE_DIR}/flow/shaders)
set(SHADER_OUTPUT_DIR ${CMAKE_CURRENT_SOURCE_DIR}/flow/shaders)
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
