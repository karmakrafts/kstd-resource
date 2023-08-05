cmake_minimum_required(VERSION 3.18)
project(kstd-resource LANGUAGES C CXX)

option(KSTD_RESOURCE_BUILD_TESTS "Build unit tests for kstd-resource" OFF)
set(KSTD_RESOURCE_MODULE ON) # Let other modules know we are present!
add_compile_definitions(KSTD_RESOURCE_MODULE)

if(NOT DEFINED CMAKE_CXX_STANDARD)
    message(STATUS "No C++ version specified, defaulting to C++20")
    set(CMAKE_CXX_STANDARD 20)
endif()

set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake;")
include(CommonLibraries)
include(Platform)
include(KstdCore)
include(KstdPlatform)

file(GLOB_RECURSE KSTD_RESOURCE_SOURCE_FILES "${CMAKE_SOURCE_DIR}/src/*.cpp")

# Shared
add_library(kstd-resource SHARED ${KSTD_RESOURCE_SOURCE_FILES})
target_include_directories(kstd-resource PUBLIC "${CMAKE_SOURCE_DIR}/include")
target_include_fmt(kstd-resource)
target_include_kstd_core(kstd-resource)
target_include_kstd_platform(kstd-resource)

# Static
add_library(kstd-resource-static STATIC ${KSTD_RESOURCE_SOURCE_FILES})
target_include_directories(kstd-resource-static PUBLIC "${CMAKE_SOURCE_DIR}/include")
target_include_fmt(kstd-resource-static)
target_include_kstd_core(kstd-resource-static)
target_include_kstd_platform(kstd-resource-static)

# Application
add_executable(kstd-rc ${KSTD_RESOURCE_SOURCE_FILES})
target_include_directories(kstd-rc PUBLIC "${CMAKE_SOURCE_DIR}/include")
target_include_fmt(kstd-rc)
target_include_kstd_core(kstd-rc)
target_include_kstd_platform(kstd-rc)
target_link_libraries(kstd-rc kstd-resource-static)
add_dependencies(kstd-rc kstd-resource-static)
target_compile_definitions(kstd-rc PUBLIC KSTD_RC_MAIN)

# Tests
if(KSTD_RESOURCE_BUILD_TESTS)
    file(GLOB_RECURSE KSTD_RESOURCE_TEST_SOURCE_FILES "${CMAKE_SOURCE_DIR}/test/*.cpp")
    add_executable(kstd-resource-tests ${KSTD_RESOURCE_TEST_SOURCE_FILES})
    target_include_directories(kstd-resource-tests PUBLIC "${CMAKE_SOURCE_DIR}/include")
    target_include_fmt(kstd-resource-tests)
    target_include_gtest(kstd-resource-tests)
    target_include_kstd_core(kstd-resource-tests)
    target_include_kstd_platform(kstd-resource-tests)
    target_link_libraries(kstd-resource-tests kstd-resource-static)
    add_dependencies(kstd-resource-tests kstd-resource-static)
endif()

if(PLATFORM_LINUX)
    target_link_libraries(kstd-resource -ldl -lrt)
    target_link_libraries(kstd-resource-static -ldl -lrt)
    if(KSTD_RESOURCE_BUILD_TESTS)
        target_link_libraries(kstd-resource-tests -ldl -lrt)
    endif()
endif()