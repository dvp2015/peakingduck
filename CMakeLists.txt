cmake_minimum_required(VERSION 3.2 FATAL_ERROR)

PROJECT(peakingduck LANGUAGES CXX)

# cmake options
OPTION(BUILD_TESTS "Build unit tests" ON)
OPTION(BUILD_DOCS "Build the documentation" OFF)

set (PROJECT_DIR_BASE ${CMAKE_CURRENT_LIST_DIR}/include)
set (TESTS_DIR_BASE ${CMAKE_CURRENT_LIST_DIR}/tests)
set (THIRD_PARTY_DIR ${CMAKE_CURRENT_LIST_DIR}/thirdparty)
set (PROJECT_BUILD_DIR ${CMAKE_BINARY_DIR})

# catch2 is header only
set (CATCH2_INCLUDES ${THIRD_PARTY_DIR}/Catch2/single_include/catch2)

# pybind11 is header only
set (PYBIND_INCLUDES ${THIRD_PARTY_DIR}/pybind11/include)

# set the output directories
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY bin)

# Get the fispact version, set in .VERSION file
file ( STRINGS "${CMAKE_SOURCE_DIR}/.VERSION" VERSION )
string( REPLACE "." ";" VERSION_LIST ${VERSION} )
list(GET VERSION_LIST 0 VERSION_MAJOR)
list(GET VERSION_LIST 1 VERSION_MINOR)
list(GET VERSION_LIST 2 VERSION_PATCH)
set(PEAKINGDUCK_VERSION "${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH}")
message ( STATUS "CMake build configuration for PEAKING DUCK ${PEAKINGDUCK_VERSION}" )

# C++ flags
SET(CMAKE_CXX_STANDARD 14)
SET(CMAKE_CXX_STANDARD_REQUIRED TRUE)

# header-only library target. To use this project as a subdirectory,
# add the following to your code:
#
# add_subdirectory(peakingduck) # or whatever you named the directory
# target_link_libraries(${PROJECT_NAME} peakingduck)
add_library(${PROJECT_NAME} INTERFACE)

target_include_directories(${PROJECT_NAME} 
	INTERFACE $<BUILD_INTERFACE:${PROJECT_DIR_BASE}> 
	$<INSTALL_INTERFACE:include> 
)

if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
	# enable parallel build
	set( ENV{CL} /MP )
	target_compile_definitions(${PROJECT_NAME} INTERFACE "/permissive-")
elseif("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
	# Increase warning levels
	add_compile_options(-Wall -Wextra -pedantic)
endif()

# unit tests
if(BUILD_TESTS)
	add_subdirectory(${TESTS_DIR_BASE}/cpp)
endif(BUILD_TESTS)

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/peakingduckConfigVersion.cmake"
  VERSION ${PEAKINGDUCK_VERSION}
  COMPATIBILITY SameMajorVersion
)

install(TARGETS peakingduck
    EXPORT peakingduckConfig
)

install(EXPORT peakingduckConfig
    NAMESPACE peakingduck::
    DESTINATION share/peakingduck/cmake
)

install(
    FILES "${CMAKE_CURRENT_BINARY_DIR}/peakingduckConfigVersion.cmake"
    DESTINATION share/units/cmake
)

install(DIRECTORY include/
    DESTINATION include
)