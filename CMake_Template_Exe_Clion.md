```cmake
cmake_minimum_required(VERSION ${CMAKE_MAJOR_VERSION}.${CMAKE_MINOR_VERSION})

project(${PROJECT_NAME}
        VERSION 0.1.0
        LANGUAGES CXX
)

# ---------------- GLOBAL SETTINGS ----------------
# Require a modern C++ version without using CMAKE_CXX_STANDARD directly
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Build options
option(ENABLE_WARNINGS_AS_ERRORS "Treat warnings as errors" OFF)
option(ENABLE_UNITY "Enable unity/jumbo builds for faster compilation" OFF)
option(ENABLE_SANITIZERS "Enable Address/Undefined sanitizers for Clang/GCC (non-MSVC)" OFF)

# ---------------- SOURCE COLLECTION ----------------
set(PROJECT_SOURCES
        # Add source files here
        # src/my_class.cpp
        ${CMAKE_DEFAULT_PROJECT_FILE}
)

set(PROJECT_HEADERS
        # Add header files here
        # include/my_class.hpp
)

# Create the executable
add_executable(${PROJECT_NAME})

# Use modern FILE_SET to group headers
target_sources(${PROJECT_NAME}
        PRIVATE ${PROJECT_SOURCES}
        PUBLIC
        FILE_SET HEADERS
        BASE_DIRS include
        FILES ${PROJECT_HEADERS}
)

target_include_directories(${PROJECT_NAME}
        PRIVATE
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

# Require standard from target side (modern style)
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

# Example for dependencies (if needed)
# find_package(fmt CONFIG REQUIRED)
# target_link_libraries(${PROJECT_NAME} PUBLIC fmt::fmt)

# Example properties
set_target_properties(${PROJECT_NAME} PROPERTIES
        VERSION ${PROJECT_VERSION}
        SOVERSION ${PROJECT_VERSION_MAJOR}
        EXPORT_NAME ${PROJECT_NAME}
)

# Copy runtime dependencies (DLLs/.so/.dylib) next to the executable after build
# Requires CMake 3.21+ for TARGET_RUNTIME_DLLS
if (CMAKE_VERSION VERSION_LESS 3.21)
    message(WARNING "Copying runtime libraries next to the executable requires CMake 3.21+. Skipping.")
else ()
    add_custom_command(TARGET mc_clone POST_BUILD
            COMMAND ${CMAKE_COMMAND} -E
            $<IF:$<BOOL:$<TARGET_RUNTIME_DLLS:${PROJECT_NAME}>>,copy_if_different,echo>
            # Sources list (only present if there are DLLs)
            $<$<BOOL:$<TARGET_RUNTIME_DLLS:${PROJECT_NAME}>>:$<TARGET_RUNTIME_DLLS:${PROJECT_NAME}>>
            # If DLLs exist: destination dir; else: a friendly message
            $<IF:$<BOOL:$<TARGET_RUNTIME_DLLS:${PROJECT_NAME}>>,$<TARGET_FILE_DIR:${PROJECT_NAME}>,"No runtime deps for ${PROJECT_NAME}">
            COMMAND_EXPAND_LISTS
    )
endif ()

```
