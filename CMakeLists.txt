cmake_minimum_required(VERSION 3.10)
project(template_matcher)

# Define that we are building a DLL
add_definitions(-DBUILDING_DLL)

if(UNIX)
    # Unix - pkg-config just works
    find_package(PkgConfig REQUIRED)
    pkg_check_modules(OpenCV REQUIRED opencv4)
    
    # Set visibility flags for Unix
    set(CMAKE_CXX_VISIBILITY_PRESET hidden)
    set(CMAKE_VISIBILITY_INLINES_HIDDEN 1)
else()
    # Windows
    message(STATUS "OPENCV_DIR environment variable: $ENV{OPENCV_DIR}")
    set(OpenCV_DIR "$ENV{OPENCV_DIR}/build")
    message(STATUS "Looking for OpenCV CMake files at: ${OpenCV_DIR}")
    if(NOT EXISTS ${OpenCV_DIR})
        message(FATAL_ERROR "OpenCV CMake files not found at ${OpenCV_DIR}. Please check your OPENCV_DIR environment variable.")
    endif()
    find_package(OpenCV REQUIRED)
endif()

message(STATUS "OpenCV library status:")
message(STATUS "    version: ${OpenCV_VERSION}")
message(STATUS "    libraries: ${OpenCV_LIBRARIES}")
message(STATUS "    include path: ${OpenCV_INCLUDE_DIRS}")

add_library(template_matcher SHARED src/cpp/template_matcher.cpp)
target_include_directories(template_matcher PRIVATE ${OpenCV_INCLUDE_DIRS})
target_link_libraries(template_matcher ${OpenCV_LIBRARIES})

set_target_properties(template_matcher PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin"
    LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib"
    ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib"
) 