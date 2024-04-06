cmake_minimum_required(VERSION 3.15)
set(PROJECT_NAME Example_Project_Template)
project(${PROJECT_NAME} VERSION 0.1.0 LANGUAGES C CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_FLAGS "-Wno-dev")
set(CMAKE_CXX_FLAGS "-fno-exceptions" "-fsanitize=address" "-Wall" "-Wextra")
set(CMAKE_CXX_FLAGS "-std=c++20")
set(PROJECT_INCLUDE_DIR TemplateProject)

add_compile_options(-fsanitize=address)
add_link_options(-fsanitize=address)

# CMake that handles precompiled headers
include(cmake/precompiled_headers.cmake)

set(PROJECT_DIR "${PROJECT_NAME}")

# Setting include directories and where to find the header files
include_directories(
    ${PROJECT_SOURCE_DIR}/TemplateProject/include
)

set(
    all_src
    ${PROJECT_SOURCE_DIR}/${PROJECT_INCLUDE_DIR}/src/${PROJECT_INCLUDE_DIR}/Core.cpp
)

add_subdirectory(Sandbox)

add_precompiled_header(${PROJECT_NAME} include/PrecompiledHeader.h FORCEINCLUDE SOURCE_CXX src/PrecompiledHeader.cpp)

add_library(${PROJECT_NAME} SHARED ${all_src})
add_library(Engine::Engine ALIAS ${PROJECT_NAME}) # Allows to alias the library

set_property(TARGET ${PROJECT_NAME} PROPERTY INTERPROCEDURAL_OPTIMIZATION True)



# Before we copy compile_commands.json to .vscode
# Making sure we should make sure if that dir already exists in the root of the projects directory
set(VSCODE_DIR "${CMAKE_CURRENT_LIST_DIR}/.vscode")
if(NOT EXISTS ${VSCODE_DIR})
    file(MAKE_DIRECTORY ${VSCODE_DIR})
endif()

# Copy to source directory
add_custom_target(
    copy-compile-commands ALL
    DEPENDS
    ${CMAKE_CURRENT_LIST_DIR}/compile_commands.json
)

# Creating a customize command specific for copying the compile_commands.json to users .vscode directory
add_custom_command(
    OUTPUT ${CMAKE_CURRENT_LIST_DIR}/compile_commands.json
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
        ${CMAKE_BINARY_DIR}/compile_commands.json
        ${CMAKE_CURRENT_LIST_DIR}/.vscode/compile_commands.json
    DEPENDS
    # Unlike "proper" targets like executables and libraries, 
    # custom command / target pairs will not set up source
    # file dependencies, so we need to list file explicitly here
    generate-compile-commands
    ${CMAKE_BINARY_DIR}/compile_commands.json
)

# Generate the compilation commands. Necessary so cmake knows where it came
# from and if for some reason you delete it.
add_custom_target(generate-compile-commands
    DEPENDS
        ${CMAKE_BINARY_DIR}/compile_commands.json
)

# add_custom_command(
#     OUTPUT ${CMAKE_BINARY_DIR}/compile_commands.json
#     COMMAND ${CMAKE_COMMAND} -B${CMAKE_BINARY_DIR} -S${CMAKE_SOURCE_DIR}
# )


# Linking the included directories
target_include_directories(
  ${PROJECT_NAME}
  PUBLIC
  ${PROJECT_SOURCE_DIR}/${PROJECT_DIR}
)

# This is how you can have precompiled headers
# This allows for faster compile time
target_precompile_headers(${PROJECT_NAME} PRIVATE
  "$<$<COMPILE_LANGUAGE:CXX>:${CMAKE_CURRENT_SOURCE_DIR}/TemplateProject/include/PrecompiledHeaders.h>"
)
