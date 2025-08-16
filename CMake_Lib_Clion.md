```cmake
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
option(${PROJECT_NAME}_BUILD_SHARED "Build shared library" OFF)
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

# Also check root directory for sources (excluding main.cpp for library)
file(GLOB ROOT_SOURCES CONFIGURE_DEPENDS
        "${CMAKE_CURRENT_SOURCE_DIR}/*.cpp"
        "${CMAKE_CURRENT_SOURCE_DIR}/*.cxx"
        "${CMAKE_CURRENT_SOURCE_DIR}/*.cc"
        "${CMAKE_CURRENT_SOURCE_DIR}/*.c"
)

# Remove main.cpp from library sources if it exists in root
list(REMOVE_ITEM ROOT_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/main.cpp")
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

# Determine library type using CLion variable with fallback
if (DEFINED CMAKE_LIBRARY_TYPE)
    set(LIB_TYPE ${CMAKE_LIBRARY_TYPE})
elseif (${PROJECT_NAME}_BUILD_SHARED)
    set(LIB_TYPE SHARED)
else ()
    set(LIB_TYPE STATIC)
endif ()

# Create the main library target using CLion variables
add_library(${PROJECT_NAME} ${LIB_TYPE} ${SOURCES} ${HEADERS})

# Set target properties
set_target_properties(${PROJECT_NAME} PROPERTIES
        VERSION ${PROJECT_VERSION}
        SOVERSION ${PROJECT_VERSION_MAJOR}
        DEBUG_POSTFIX "_d"
)

# Handle shared library exports
if (LIB_TYPE STREQUAL "SHARED")
    target_compile_definitions(${PROJECT_NAME} PRIVATE ${PROJECT_NAME}_EXPORTS)
    if (WIN32)
        target_compile_definitions(${PROJECT_NAME} PUBLIC ${PROJECT_NAME}_DLL)
    endif ()
endif ()

# Include directories for the target
target_include_directories(${PROJECT_NAME}
        PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>
        PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/src
        ${CMAKE_CURRENT_SOURCE_DIR}
)

# Link libraries (uncomment and modify as needed)
# find_package(Threads REQUIRED)
# target_link_libraries(${PROJECT_NAME} PRIVATE Threads::Threads)

# Create alias target for easier use in parent projects
add_library(${PROJECT_NAME}::${PROJECT_NAME} ALIAS ${PROJECT_NAME})

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

    # Install the library
    install(TARGETS ${PROJECT_NAME}
            EXPORT ${PROJECT_NAME}Targets
            LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
            ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
            RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
            INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
    )

    # Install executable if it exists
    if (TARGET ${PROJECT_NAME}_exe)
        install(TARGETS ${PROJECT_NAME}_exe
                RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        )
    endif ()

    # Install headers
    if (EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/include")
        install(DIRECTORY include/
                DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
                FILES_MATCHING PATTERN "*.h" PATTERN "*.hpp"
        )
    endif ()

    # Export targets
    install(EXPORT ${PROJECT_NAME}Targets
            FILE ${PROJECT_NAME}Targets.cmake
            NAMESPACE ${PROJECT_NAME}::
            DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
    )

    # Create basic config file
    set(CONFIG_FILE_CONTENT "
include(CMakeFindDependencyMacro)

# Add any dependencies here
# find_dependency(SomePackage)

include(\"\${CMAKE_CURRENT_LIST_DIR}/${PROJECT_NAME}Targets.cmake\")
")
    file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake" "${CONFIG_FILE_CONTENT}")

    # Create version file
    write_basic_package_version_file(
            "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
            VERSION ${PROJECT_VERSION}
            COMPATIBILITY AnyNewerVersion
    )

    # Install config files
    install(FILES
            "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
            "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
            DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
    )
endif ()

# Print summary
message(STATUS "")
message(STATUS "Configuration Summary:")
message(STATUS "  Project: ${PROJECT_NAME} ${PROJECT_VERSION}")
message(STATUS "  C++ Standard: ${CMAKE_CXX_STANDARD}")
message(STATUS "  Build Type: ${CMAKE_BUILD_TYPE}")
message(STATUS "  Library Type: ${LIB_TYPE}")
message(STATUS "  Build Tests: ${${PROJECT_NAME}_BUILD_TESTS}")
message(STATUS "  Build Examples: ${${PROJECT_NAME}_BUILD_EXAMPLES}")
message(STATUS "  Install: ${${PROJECT_NAME}_INSTALL}")
message(STATUS "")
```
