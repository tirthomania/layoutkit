cmake_minimum_required(VERSION 3.20)
project(layoutkit LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)   # lets VS Code / clangd understand the project

option(LAYOUTKIT_USE_SOLUTIONS "Build the reference solutions instead of your code" OFF)
option(LAYOUTKIT_SANITIZE "Build with AddressSanitizer + UndefinedBehaviorSanitizer (GCC/Clang)" OFF)

if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  set(CMAKE_BUILD_TYPE Debug CACHE STRING "Build type" FORCE)
endif()

if(MSVC)
  add_compile_options(/W4 /permissive-)
else()
  add_compile_options(-Wall -Wextra -Wpedantic -Wshadow)
endif()

if(LAYOUTKIT_SANITIZE AND NOT MSVC)
  add_compile_options(-fsanitize=address,undefined -fno-omit-frame-pointer -g)
  add_link_options(-fsanitize=address,undefined)
endif()

# ---------------------------------------------------------------------------
# The library: your code in src/, or the reference solutions in solutions/src/
# ---------------------------------------------------------------------------
if(LAYOUTKIT_USE_SOLUTIONS)
  set(LK_SRC_DIR ${CMAKE_CURRENT_SOURCE_DIR}/solutions/src)
  message(STATUS "layoutkit: building the REFERENCE SOLUTIONS")
else()
  set(LK_SRC_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src)
endif()

add_library(layoutkit STATIC
  ${LK_SRC_DIR}/geometry.cpp
  ${LK_SRC_DIR}/polygon.cpp
  ${LK_SRC_DIR}/layout.cpp
  ${LK_SRC_DIR}/spatial_index.cpp
  ${LK_SRC_DIR}/connectivity.cpp
  ${LK_SRC_DIR}/maze_router.cpp
)

# Header-only exercises (M4 templates): solutions/include shadows include/ when enabled.
if(LAYOUTKIT_USE_SOLUTIONS)
  target_include_directories(layoutkit PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/solutions/include)
endif()
target_include_directories(layoutkit PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include)

# ---------------------------------------------------------------------------
# Tests: one executable per milestone. Run one with:  ctest --test-dir build -R m1
# ---------------------------------------------------------------------------
enable_testing()
set(LK_MILESTONES
  m1_geometry
  m2_polygon
  m3_layout
  m4_interval
  m5_spatial_index
  m6_connectivity
  m7_router
)
foreach(t IN LISTS LK_MILESTONES)
  add_executable(test_${t} tests/test_${t}.cpp tests/check_main.cpp)
  target_include_directories(test_${t} PRIVATE tests)
  target_link_libraries(test_${t} PRIVATE layoutkit)
  add_test(NAME ${t} COMMAND test_${t})
endforeach()

# ---------------------------------------------------------------------------
# Benchmark: brute force vs your GridIndex. Build in Release to get real numbers.
# ---------------------------------------------------------------------------
add_executable(bench_index bench/bench_index.cpp)
target_link_libraries(bench_index PRIVATE layoutkit)
