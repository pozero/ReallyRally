cmake_minimum_required(VERSION 3.16)

project(ReallyRally
    LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON CACHE INTERNAL "")

find_program(CCACH_PATH ccache)
if(CCACH_PATH)
    message(STATUS "Using ccache at ${CCACH_PATH}")
    set(CMAKE_CXX_COMPILER_LAUNCHER ${CCACH_PATH})
else()
    message(STATUS "Can't find ccache")
endif()

option(IPO "Enable linke time optimization (might significantly increase build time)" ON)
include(CheckIPOSupported)
check_ipo_supported(RESULT IPO_SUPPORTED OUTPUT IPO_ERROR)
if(IPO_SUPPORTED)
    message(STATUS "IPO: ${IPO}")
else()
    set(IPO OFF)
    message(STATUS "IPO isn't supported: <${IPO_ERROR}>")
endif()

add_executable(ReallyRally)

if (IPO)
    set_property(TARGET ReallyRally PROPERTY INTERPROCEDURAL_OPTIMIZATION TRUE)
endif()

target_compile_definitions(ReallyRally
    PRIVATE
        ROOT_PATH="${PROJECT_SOURCE_DIR}/"
        BINARY_PATH="${PROJECT_BINARY_DIR}/"
)

if(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
    target_compile_options(ReallyRally 
        PRIVATE 
            -Werror
            -Weverything
            -Weffc++
            -Wno-c++98-compat
            -Wno-c++98-compat-pedantic
            -Wno-c++11-compat
            -Wno-c++14-compat
            -Wno-c++17-compat
            -Wno-c++20-compat
            -Wno-c++20-extensions
            -Wno-old-style-cast
            -Wno-newline-eof
            -Wno-padded
            -ffast-math 
            $<$<CONFIG:Debug>:-fstandalone-debug>
            $<$<CONFIG:RelWithDebInfo>:-fstandalone-debug>
    )
else()
    message(FATAL_ERROR "Only support Clang toolchain")
endif()

file(GLOB_RECURSE 
   CPP_SOURCE_FILE
       "${PROJECT_SOURCE_DIR}/src/*.cpp"
       "${PROJECT_SOURCE_DIR}/src/*.h"
)

target_sources(ReallyRally
    PRIVATE
        ${CPP_SOURCE_FILE}
)