```cmake
# Modern CMake Template for CLion
# Uses CLion template variables: ${CMAKE_MAJOR_VERSION}, ${CMAKE_MINOR_VERSION}, ${PROJECT_NAME}, etc.

# Minimum CMake version using CLion variables
cmake_minimum_required(VERSION ${CMAKE_MAJOR_VERSION}.${CMAKE_MINOR_VERSION})

# Project declaration using CLion template variable
project(${PROJECT_NAME}
        VERSION 1.0.0
        DESCRIPTION "A modern C++ project"
        LANGUAGES CXX
)

# C++ standard using CLion variable with fallback
if (DEFINED CMAKE_LANGUAGE_VERSION)
    set(CMAKE_CXX_STANDARD ${CMAKE_LANGUAGE_VERSION})
else ()
    set(CMAKE_CXX_STANDARD 20)
endif ()

# Enforce the C++ standard (no extensions, required)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Generate compile_commands.json for IDE support
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Organize targets in IDE folders
set_property(GLOBAL PROPERTY USE_FOLDERS ON)

# Build type handling
if (NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    message(STATUS "Setting build type to 'Release' as none was specified.")
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build." FORCE)
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif ()

# Options for the project using CLion project name variable
option(${PROJECT_NAME}_BUILD_TESTS "Build tests" ON)
option(${PROJECT_NAME}_BUILD_EXAMPLES "Build examples" ON)
option(${PROJECT_NAME}_INSTALL "Generate install target" ON)

# Compiler-specific options
if (MSVC)
    add_compile_options(/W4 /permissive-)
    if (CMAKE_BUILD_TYPE STREQUAL "Debug")
        add_compile_options(/Od /Zi)
    endif ()
else ()
    add_compile_options(-Wall -Wextra -Wpedantic)
    if (CMAKE_BUILD_TYPE STREQUAL "Debug")
        add_compile_options(-g -O0)
    endif ()
endif ()

# Debug vs Release definitions
if (CMAKE_BUILD_TYPE STREQUAL "Debug")
    add_compile_definitions(DEBUG)
endif ()

# Include directories
include_directories(
        ${CMAKE_CURRENT_SOURCE_DIR}/include
        ${CMAKE_CURRENT_SOURCE_DIR}/src
)

# Always auto-discover source files using GLOB_RECURSE with CONFIGURE_DEPENDS
file(GLOB_RECURSE SOURCES CONFIGURE_DEPENDS
        "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp"
        "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cxx"
        "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cc"
        "${CMAKE_CURRENT_SOURCE_DIR}/src/*.c"
)

# Also check root directory for sources
file(GLOB ROOT_SOURCES CONFIGURE_DEPENDS
        "${CMAKE_CURRENT_SOURCE_DIR}/*.cpp"
        "${CMAKE_CURRENT_SOURCE_DIR}/*.cxx"
        "${CMAKE_CURRENT_SOURCE_DIR}/*.cc"
        "${CMAKE_CURRENT_SOURCE_DIR}/*.c"
)

# include all sources including main.cpp
list(APPEND SOURCES ${ROOT_SOURCES})

# Handle CLion's default project file if specified (add to sources)
if (DEFINED CMAKE_DEFAULT_PROJECT_FILE AND EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/${CMAKE_DEFAULT_PROJECT_FILE}")
    list(APPEND SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/${CMAKE_DEFAULT_PROJECT_FILE}")
endif ()

# Always auto-discover header files using GLOB_RECURSE with CONFIGURE_DEPENDS
file(GLOB_RECURSE HEADERS CONFIGURE_DEPENDS
        "${CMAKE_CURRENT_SOURCE_DIR}/include/*.h"
        "${CMAKE_CURRENT_SOURCE_DIR}/include/*.hpp"
        "${CMAKE_CURRENT_SOURCE_DIR}/include/*.hxx"
        "${CMAKE_CURRENT_SOURCE_DIR}/src/*.h"
        "${CMAKE_CURRENT_SOURCE_DIR}/src/*.hpp"
        "${CMAKE_CURRENT_SOURCE_DIR}/src/*.hxx"
        "${CMAKE_CURRENT_SOURCE_DIR}/*.h"
        "${CMAKE_CURRENT_SOURCE_DIR}/*.hpp"
        "${CMAKE_CURRENT_SOURCE_DIR}/*.hxx"
)

# Create a placeholder header if none exist
if (NOT HEADERS AND SOURCES)
    file(MAKE_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/include")
    set(PLACEHOLDER_HEADER "${CMAKE_CURRENT_SOURCE_DIR}/include/${PROJECT_NAME}.hpp")
    if (NOT EXISTS "${PLACEHOLDER_HEADER}")
        file(WRITE "${PLACEHOLDER_HEADER}"
                "#pragma once\n\n"
                "namespace ${PROJECT_NAME} {\n"
                "    void hello();\n"
                "}\n"
        )
    endif ()
    list(APPEND HEADERS "${PLACEHOLDER_HEADER}")
endif ()


# Create executable target
add_executable(${PROJECT_NAME} ${SOURCES} ${HEADERS})

# Set executable properties
set_target_properties(${PROJECT_NAME} PROPERTIES
        DEBUG_POSTFIX "_d"
)

# Include directories for the target
target_include_directories(${PROJECT_NAME}
        PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/include
        ${CMAKE_CURRENT_SOURCE_DIR}/src
        ${CMAKE_CURRENT_SOURCE_DIR}
)

# Link libraries (uncomment and modify as needed)
# find_package(Threads REQUIRED)
# target_link_libraries(${PROJECT_NAME} PRIVATE Threads::Threads)

# Tests
if (${PROJECT_NAME}_BUILD_TESTS AND EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/tests")
    enable_testing()
    add_subdirectory(tests)
endif ()

# Examples
if (${PROJECT_NAME}_BUILD_EXAMPLES AND EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/examples")
    add_subdirectory(examples)
endif ()

# Installation
if (${PROJECT_NAME}_INSTALL)
    include(GNUInstallDirs)
    include(CMakePackageConfigHelpers)

    # Install the main target
    install(TARGETS ${PROJECT_NAME}
            RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    )
endif ()

# Print summary
message(STATUS "")
message(STATUS "Configuration Summary:")
message(STATUS "  Project: ${PROJECT_NAME} ${PROJECT_VERSION}")
message(STATUS "  C++ Standard: ${CMAKE_CXX_STANDARD}")
message(STATUS "  Build Type: ${CMAKE_BUILD_TYPE}")
message(STATUS "  Build Tests: ${${PROJECT_NAME}_BUILD_TESTS}")
message(STATUS "  Build Examples: ${${PROJECT_NAME}_BUILD_EXAMPLES}")
message(STATUS "  Install: ${${PROJECT_NAME}_INSTALL}")
message(STATUS "")
```
