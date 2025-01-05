cmake_minimum_required(VERSION 3.10)
project(template_matcher)

# Set C++11 as required
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Define that we are building a DLL
add_definitions(-DBUILDING_DLL)

if(UNIX)
    # Unix - pkg-config just works
    find_package(PkgConfig REQUIRED)
    pkg_check_modules(OpenCV REQUIRED opencv4)
    
    # Set visibility flags for Unix
    set(CMAKE_CXX_VISIBILITY_PRESET hidden)
    set(CMAKE_VISIBILITY_INLINES_HIDDEN 1)

    if(APPLE)
        # Debug output for macOS
        message(STATUS "macOS OpenCV library paths:")
        execute_process(
            COMMAND ls -la $ENV{OPENCV_LIB_DIR}
            COMMAND grep "libopencv"
            OUTPUT_VARIABLE OPENCV_LIB_LIST
        )
        message(STATUS "Found libraries:\n${OPENCV_LIB_LIST}")

        # On macOS, libraries are named libopencv_*.dylib
        set(OpenCV_LIBRARIES "")
        foreach(lib ${OpenCV_LIBRARIES})
            string(REGEX REPLACE "^opencv_" "libopencv_" DYLIB_NAME "${lib}")
            string(APPEND DYLIB_NAME ".dylib")
            list(APPEND OpenCV_LIBRARIES "$ENV{OPENCV_LIB_DIR}/${DYLIB_NAME}")
        endforeach()
    endif()
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

# Add debug output for each library
foreach(lib ${OpenCV_LIBRARIES})
    message(STATUS "    checking library: ${lib}")
    if(EXISTS "${lib}")
        message(STATUS "    ${lib} exists")
    else()
        message(STATUS "    ${lib} not found")
    endif()
endforeach()

add_library(template_matcher SHARED src/cpp/template_matcher.cpp)
target_include_directories(template_matcher PRIVATE ${OpenCV_INCLUDE_DIRS})
target_link_libraries(template_matcher ${OpenCV_LIBRARIES})

set_target_properties(template_matcher PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin"
    LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib"
    ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib"
) 