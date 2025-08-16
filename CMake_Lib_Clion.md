```cmake
cmake_minimum_required(VERSION ${CMAKE_MAJOR_VERSION}.${CMAKE_MINOR_VERSION})

project(${PROJECT_NAME}
        VERSION 0.1.0
        LANGUAGES CXX
)

# ---------------- GLOBAL SETTINGS ----------------
# Enforce modern C++
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Build options
option(BUILD_SHARED_LIBS "Build shared libraries" ON)
option(ENABLE_WARNINGS_AS_ERRORS "Treat warnings as errors" OFF)
option(ENABLE_UNITY "Enable unity/jumbo builds for faster compilation" OFF)
option(ENABLE_SANITIZERS "Enable Address/Undefined sanitizers for Clang/GCC (non-MSVC)" OFF)

# Organize targets in IDEs (CLion, VS, Xcode, etc.)
#set_property(GLOBAL PROPERTY USE_FOLDERS ON)

# ---------------- SOURCE COLLECTION ----------------
file(GLOB_RECURSE PROJECT_SOURCES CONFIGURE_DEPENDS
        src/*.cpp
        src/*.cxx
        src/*.cc
)

file(GLOB_RECURSE PROJECT_HEADERS CONFIGURE_DEPENDS
        include/*.hpp
        include/*.h
)

# ---------------- LIBRARY ----------------
add_library(${PROJECT_NAME} ${CMAKE_LIBRARY_TYPE})
set_target_properties(${PROJECT_NAME} PROPERTIES POSITION_INDEPENDENT_CODE ON)

target_sources(${PROJECT_NAME}
        PRIVATE ${PROJECT_SOURCES}
        PUBLIC
        FILE_SET HEADERS TYPE HEADERS
        BASE_DIRS include
        FILES ${PROJECT_HEADERS}
)

# Modern way to enforce language standard
target_compile_features(${PROJECT_NAME} PUBLIC cxx_std_${CMAKE_LANGUAGE_VERSION})

# Warnings (per-compiler)
if (MSVC)
    target_compile_options(${PROJECT_NAME} PRIVATE /W4 /permissive- /Zc:__cplusplus)
else ()
    target_compile_options(${PROJECT_NAME} PRIVATE -Wall -Wextra -Wpedantic)
    if (CMAKE_CXX_COMPILER_ID MATCHES "Clang|GNU")
        target_compile_options(${PROJECT_NAME} PRIVATE -Wconversion -Wsign-conversion)
    endif ()
endif ()

if (ENABLE_WARNINGS_AS_ERRORS)
    if (MSVC)
        target_compile_options(${PROJECT_NAME} PRIVATE /WX)
    else ()
        target_compile_options(${PROJECT_NAME} PRIVATE -Werror)
    endif ()
endif ()

# Unity build (CMake >= 3.16)
if (ENABLE_UNITY)
    set_target_properties(${PROJECT_NAME} PROPERTIES UNITY_BUILD ON)
endif ()

# Optional sanitizers (Debug-only recommended)
if (ENABLE_SANITIZERS AND NOT MSVC)
    target_compile_options(${PROJECT_NAME} PRIVATE -fsanitize=address,undefined -fno-omit-frame-pointer)
    target_link_options(${PROJECT_NAME} PRIVATE -fsanitize=address,undefined)
endif ()


# Generate export header
include(GenerateExportHeader)
generate_export_header(${PROJECT_NAME}
        EXPORT_FILE_NAME ${CMAKE_CURRENT_BINARY_DIR}/include/${PROJECT_NAME}_export.h
)

# Correctly expose include dirs (build + install)
target_include_directories(${PROJECT_NAME}
        PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/include>
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

# Example external dependency (optional)
# find_package(fmt CONFIG REQUIRED)
# target_link_libraries(${PROJECT_NAME} PUBLIC fmt::fmt)

# Create namespaced ALIAS for consumers and consistency with exports
add_library(${PROJECT_NAME}::${PROJECT_NAME} ALIAS ${PROJECT_NAME})

# Properties for versioning
set_target_properties(${PROJECT_NAME} PROPERTIES
        VERSION ${PROJECT_VERSION}
        SOVERSION ${PROJECT_VERSION_MAJOR}
        EXPORT_NAME ${PROJECT_NAME}
        # FOLDER "Libraries" # shows up as 'Libraries/MyLib' in IDEs
)

# ---------------- INSTALL / EXPORT ----------------
include(GNUInstallDirs)

# Only install/export when this is the top-level project
if (PROJECT_IS_TOP_LEVEL)
    install(TARGETS ${PROJECT_NAME}
            EXPORT ${PROJECT_NAME}Targets
            FILE_SET HEADERS
            RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
            LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
            ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
            INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
    )

    install(EXPORT ${PROJECT_NAME}Targets
            NAMESPACE ${PROJECT_NAME}::
            DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
    )

    include(CMakePackageConfigHelpers)

    write_basic_package_version_file(
            "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
            VERSION ${PROJECT_VERSION}
            COMPATIBILITY SameMajorVersion
    )

    if (NOT EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/cmake/${PROJECT_NAME}Config.cmake.in")
        file(WRITE "${CMAKE_CURRENT_SOURCE_DIR}/cmake/${PROJECT_NAME}Config.cmake.in"
                "@PACKAGE_INIT@

include(CMakeFindDependencyMacro)

# Uncomment and modify if package has dependencies
# find_dependency(fmt CONFIG REQUIRED)

include(\"\${CMAKE_CURRENT_LIST_DIR}/@PROJECT_NAME@Targets.cmake\")

check_required_components(@PROJECT_NAME@)
")
    endif ()

    configure_package_config_file(
            "${CMAKE_CURRENT_SOURCE_DIR}/cmake/${PROJECT_NAME}Config.cmake.in"
            "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
            INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
    )

    # Install config + version files
    install(FILES
            "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
            "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
            DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
    )

    # Install generated export header to the public include tree
    install(FILES
            "${CMAKE_CURRENT_BINARY_DIR}/include/${PROJECT_NAME}_export.h"
            DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
    )
endif ()
```
