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
        # Add OpenCV library directory to link path
        link_directories(${OpenCV_LIBRARY_DIRS})
        
        # List only the libraries we need
        set(OpenCV_LIBS
            opencv_core
            opencv_imgproc
            opencv_imgcodecs
        )
        
        # Debug output
        message(STATUS "OpenCV library dirs: ${OpenCV_LIBRARY_DIRS}")
        message(STATUS "OpenCV libraries: ${OpenCV_LIBS}")
        
        # Update target link libraries
        target_link_libraries(template_matcher PRIVATE ${OpenCV_LIBS})
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