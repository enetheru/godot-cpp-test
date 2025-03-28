#[=======================================================================[.rst:
Integration Testing
-------------------

The Test target used to validate changes.
This is a duplication from the godot-cpp/test folder but separated so as
to check building from an external project.
]=======================================================================]

cmake_minimum_required( VERSION 3.17 )

#[=============================[ Global Options ]=============================]
# set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")
# set(CMAKE_OSX_ARCHITECTURES "arm64;x86_64")

#[==============================[ Dependencies ]==============================]
include( FetchContent )

set( FETCHCONTENT_FULLY_DISCONNECTED OFF )

# Python
find_package(Python3 3.4 REQUIRED) # pathlib should be present

# Godot-cpp
set( GODOTCPP_GIT_URL "http://github.com/godotengine/godot-cpp.git" CACHE STRING "The git url of godot-cpp to fetch" )
set( GODOTCPP_GIT_BRANCH "master" CACHE STRING "The git branch of godot-cpp to fetch" )
set( GODOTCPP_BUILD_PROFILE "${CMAKE_SOURCE_DIR}/build_profile.json" )
FetchContent_Declare( godot-cpp
        GIT_REPOSITORY ${GODOTCPP_GIT_URL}
        GIT_TAG ${GODOTCPP_GIT_BRANCH}
        GIT_PROGRESS ON
)
FetchContent_MakeAvailable( godot-cpp )

#[===========================[ Project Definition ]===========================]
# We add the project after the godot-cpp so that
# it can pick up on changes made in godot-cpp to the CMAKE_ variables.
project( gdexample
        VERSION 0.0.0
        DESCRIPTION "Stand-Alone godot-cpp Integration Testing"
        HOMEPAGE_URL "https://github.com/enetheru/godot-cpp-test"
        LANGUAGES CXX
)

#[================================[ Options ]================================]
set( TEST_TARGET "template_debug" CACHE STRING "Which godot-cpp::target to link against" )
set_property( CACHE TEST_TARGET PROPERTY STRINGS "template_debug;template_release;editor" )

# Get useful properties from godot-cpp
get_target_property( GODOTCPP_SUFFIX godot-cpp GODOTCPP_SUFFIX )

#[=============================[ Configuration ]=============================]
set( OUTPUT_NAME "${PROJECT_NAME}${GODOTCPP_SUFFIX}" )
set( OUTPUT_DIR "${CMAKE_CURRENT_SOURCE_DIR}/project/bin/" )

if( CMAKE_SYSTEM_NAME STREQUAL Darwin )
    set( OUTPUT_DIR "${OUTPUT_DIR}/lib${PROJECT_NAME}.macos.${TEST_TARGET}.framework")
endif()

#[================================[ Doc-Gen ]================================]
file( GLOB_RECURSE XML_SOURCES LIST_DIRECTORIES NO CONFIGURE_DEPENDS
        ${CMAKE_CURRENT_SOURCE_DIR}/doc_classes/*.xml )
generate_doc_source( DOC_DATA "${XML_SOURCES}" )

#[===========================[ Target Definition ]===========================]
add_library( ${PROJECT_NAME} SHARED )

target_sources( ${PROJECT_NAME}
        PRIVATE
        src/example.cpp
        src/example.h
        src/register_types.cpp
        src/register_types.h
        src/tests.h
        ${DOC_DATA}
)

target_link_libraries( ${PROJECT_NAME} PRIVATE godot-cpp )

set_target_properties( ${PROJECT_NAME}
        PROPERTIES
        CXX_STANDARD 17
        CXX_EXTENSIONS OFF
        CXX_VISIBILITY_PRESET ${GODOTCPP_SYMBOL_VISIBILITY}

        POSITION_INDEPENDENT_CODE ON
        BUILD_RPATH_USE_ORIGIN ON

        # Try to ensure only static libraries are selected to be linked to.
        LINK_SEARCH_START_STATIC ON
        LINK_SEARCH_END_STATIC ON

        # NOTE: Wrapping the output variables inside a generator expression
        # prevents msvc generator from adding addition Config Directories
        LIBRARY_OUTPUT_DIRECTORY "$<1:${OUTPUT_DIR}>"
        RUNTIME_OUTPUT_DIRECTORY "$<1:${OUTPUT_DIR}>"
        PDB_OUTPUT_DIRECTORY     "$<1:${OUTPUT_DIR}>" #MSVC Only, ignored on other platforms

        PREFIX "lib"
        OUTPUT_NAME "${OUTPUT_NAME}"
)
