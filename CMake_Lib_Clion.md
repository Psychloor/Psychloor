```cmake
cmake_minimum_required(VERSION ${CMAKE_MAJOR_VERSION}.${CMAKE_MINOR_VERSION})

project(${PROJECT_NAME}
    VERSION 0.1.0
    LANGUAGES CXX
)

# Require a modern C++ version without using CMAKE_CXX_STANDARD directly
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Collect source files (recursively)
file(GLOB_RECURSE PROJECT_SOURCES CONFIGURE_DEPENDS
    src/*.cpp
    src/*.cxx
    src/*.cc
)

file(GLOB_RECURSE PROJECT_HEADERS CONFIGURE_DEPENDS
    include/*.hpp
    include/*.h
)

# Create the library
add_library(${PROJECT_NAME} ${CMAKE_LIBRARY_TYPE})

# Use modern FILE_SET to group headers
target_sources(${PROJECT_NAME}
    PRIVATE ${PROJECT_SOURCES}
    PUBLIC
        FILE_SET HEADERS
        BASE_DIRS include
        FILES ${PROJECT_HEADERS}
)

target_include_directories(${PROJECT_NAME} PUBLIC
        include/
)

# Require standard from target side (modern style)
target_compile_features(${PROJECT_NAME} PUBLIC cxx_std_${CMAKE_LANGUAGE_VERSION})

# Example for dependencies (if needed)
# find_package(fmt CONFIG REQUIRED)
# target_link_libraries(${PROJECT_NAME} PUBLIC fmt::fmt)

# Example properties
set_target_properties(${PROJECT_NAME} PROPERTIES
    VERSION ${PROJECT_VERSION}
    SOVERSION ${PROJECT_VERSION_MAJOR}
    EXPORT_NAME ${PROJECT_NAME}
)
```
