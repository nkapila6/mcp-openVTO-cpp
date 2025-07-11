cmake_minimum_required(VERSION 3.10)
project(MCP VERSION 2024.11.05 LANGUAGES CXX)

set(CMAKE_WARN_UNUSED_CLI YES)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

if (NOT XCODE AND NOT MSVC AND NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type" FORCE)
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()

option(BUILD_SHARED_LIBS "build shared libraries" ${BUILD_SHARED_LIBS_DEFAULT})

if (WIN32)
    add_compile_definitions(_CRT_SECURE_NO_WARNINGS)
endif()

if (MSVC)
    add_compile_options("$<$<COMPILE_LANGUAGE:C>:/utf-8>")
    add_compile_options("$<$<COMPILE_LANGUAGE:CXX>:/utf-8>")
    add_compile_options("$<$<COMPILE_LANGUAGE:C>:/bigobj>")
    add_compile_options("$<$<COMPILE_LANGUAGE:CXX>:/bigobj>")
endif()

# Set C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Find required packages
find_package(Threads REQUIRED)

option(MCP_SSL "Enable SSL support" ON)
# option(MCP_SSL "Enable SSL support" OFF)

if(MCP_SSL)
    find_package(OpenSSL 3.0.0 COMPONENTS Crypto SSL REQUIRED)
    include_directories(${OPENSSL_INCLUDE_DIR}) # before including cpp-httplib to avoid OpenSSL with lower version included
    message(STATUS "OpenSSL include directory: ${OPENSSL_INCLUDE_DIR}")
    add_compile_definitions(MCP_SSL CPPHTTPLIB_OPENSSL_SUPPORT)
endif()

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/common)

# including Eigen
include(FetchContent)
FetchContent_Declare(
    Eigen3
    GIT_REPOSITORY https://gitlab.com/libeigen/eigen.git
    GIT_TAG 3.4.0
)
FetchContent_MakeAvailable(Eigen3)

add_subdirectory(src)
add_subdirectory(apps)
add_subdirectory(apps_testing)

# Add test directory
option(MCP_BUILD_TESTS "Build the tests" OFF)
if(MCP_BUILD_TESTS)
    enable_testing()
    add_subdirectory(test)
endif()
