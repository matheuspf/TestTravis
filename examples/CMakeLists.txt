cmake_minimum_required(VERSION 2.8.8)
project (examples)

get_filename_component(PARENT_DIR ${PROJECT_SOURCE_DIR} DIRECTORY)

include_directories(${PARENT_DIR}/include)


set(CMAKE_CXX_FLAGS  "${CMAKE_CXX_FLAGS} -std=c++14 -g -fprofile-arcs -ftest-coverage")
set(CMAKE_EXE_LINKER_FLAGS  "${CMAKE_EXE_LINKER_FLAGS} -lgcov")
set(CMAKE_CXX_OUTPUT_EXTENSION_REPLACE 1)


add_executable(LoopingExample LoopingExample.cpp)

add_executable(STLExample STLExample.cpp)