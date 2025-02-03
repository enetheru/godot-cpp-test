cmake_minimum_required( VERSION 3.17 )

#[[ Python is required for documentation generation ]]
find_package(Python3 3.4 REQUIRED) # pathlib should be present

set( GODOTCPP_GIT_URL "http://github.com/enetheru/godot-cpp.git" CACHE STRING
        "The git url of godot-cpp to fetch" )
set( GODOTCPP_GIT_BRANCH "master" CACHE STRING "The git branch of godot-cpp to fetch" )

project( gdexample
        VERSION 0.0.0
        DESCRIPTION ""
        HOMEPAGE_URL ""
        LANGUAGES CXX
)

include( FetchContent )

FetchContent_Declare( godot-cpp
        GIT_REPOSITORY ${GODOTCPP_GIT_URL}
        GIT_TAG ${GODOTCPP_GIT_BRANCH}
        GIT_PROGRESS ON
)
FetchContent_MakeAvailable( godot-cpp )

add_library( gdexample SHARED )

file( GLOB_RECURSE XML_SOURCES LIST_DIRECTORIES NO CONFIGURE_DEPENDS
        ${CMAKE_CURRENT_SOURCE_DIR}/doc_classes/*.xml )

# Generate doc_data.gen.cpp
generate_doc_source( DOC_DATA "${XML_SOURCES}" )

target_sources( gdexample
        PRIVATE
        src/example.cpp
        src/example.h
        src/register_types.cpp
        src/register_types.h
        src/tests.h
        ${DOC_DATA}
)

set( TEST_TARGET "template_debug" CACHE STRING "Which godot-cpp::target to link against" )
set_property( CACHE TEST_TARGET PROPERTY STRINGS "template_debug;template_release;editor" )

target_link_libraries( gdexample
        PRIVATE
        godot-cpp::${TEST_TARGET} )

### Get useful properties of the library
get_target_property( GODOT_PLATFORM godot-cpp::${TEST_TARGET} GODOT_PLATFORM )
get_target_property( GODOT_TARGET godot-cpp::${TEST_TARGET} GODOT_TARGET )
get_target_property( GODOT_ARCH godot-cpp::${TEST_TARGET} GODOT_ARCH )

set( OUTPUT_DIR "${CMAKE_CURRENT_SOURCE_DIR}/project/bin/" )

set_target_properties( gdexample
        PROPERTIES
        CXX_STANDARD 17
        CXX_EXTENSIONS OFF
        CXX_VISIBILITY_PRESET ${GODOT_SYMBOL_VISIBILITY}

        BUILD_RPATH_USE_ORIGIN ON

        LINK_SEARCH_START_STATIC ON
        LINK_SEARCH_END_STATIC ON

        # NOTE: Wrapping the output variables inside a generator expression
        # prevents msvc generator from adding addition Config Directories
        LIBRARY_OUTPUT_DIRECTORY "$<1:${OUTPUT_DIR}>"
        RUNTIME_OUTPUT_DIRECTORY "$<1:${OUTPUT_DIR}>"
        PDB_OUTPUT_DIRECTORY     "$<1:${OUTPUT_DIR}>" #MSVC Only, ignored on other platforms

        PREFIX "lib"
        OUTPUT_NAME "gdexample.${GODOT_PLATFORM}.${GODOT_TARGET}.${GODOT_ARCH}"
)

if( CMAKE_SYSTEM_NAME STREQUAL Darwin )
    get_target_property( OSX_ARCH godot-cpp::${TEST_TARGET} OSX_ARCHITECTURES )

    set( OUTPUT_DIR "${OUTPUT_DIR}/libgdexample.macos.${TEST_TARGET}.framework")

    set_target_properties( gdexample
            PROPERTIES
            OUTPUT_NAME "gdexample.macos.${TEST_TARGET}"
            SUFFIX ""

            #macos options
            OSX_ARCHITECTURES "${OSX_ARCH}"
    )
endif ()
