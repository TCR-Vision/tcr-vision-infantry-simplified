cmake_minimum_required(VERSION 3.16)
project(serial)

if(NOT CMAKE_SYSTEM_NAME STREQUAL "Linux")
    message(FATAL_ERROR "This trimmed serial library only supports Linux")
endif()

add_library(serial
    src/serial.cc
    src/impl/unix.cc
)
target_include_directories(serial PUBLIC include)
target_link_libraries(serial rt pthread)
