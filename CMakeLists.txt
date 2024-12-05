cmake_minimum_required( VERSION 3.17 )
project( godot-cpp-test
        VERSION 0.0.0
        DESCRIPTION ""
        HOMEPAGE_URL ""
        LANGUAGES CXX
)

include( FetchContent )

FetchContent_Declare( godot-cpp
        GIT_REPOSITORY https://github.com/enetheru/godot-cpp.git
        GIT_TAG "privatise"
        SYSTEM
        EXCLUDE_FROM_ALL
)

FetchContent_MakeAvailable( godot-cpp )

add_library( godot-cpp-test SHARED )

target_sources( godot-cpp-test
        PRIVATE
        src/example.cpp
        src/example.h
        src/register_types.cpp
        src/register_types.h
        src/tests.h
)

set( TEST_TARGET "template_debug" CACHE STRING "Which godot-cpp::target to link against" )
set_property( CACHE TEST_TARGET PROPERTY STRINGS "template_debug;template_release;editor" )

target_link_libraries( godot-cpp-test
        PRIVATE
        godot-cpp::${TEST_TARGET} )

### Get useful properties of the library
get_target_property( GODOT_PLATFORM godot-cpp::${TEST_TARGET} GODOT_PLATFORM )
get_target_property( GODOT_TARGET godot-cpp::${TEST_TARGET} GODOT_TARGET )
get_target_property( GODOT_ARCH godot-cpp::${TEST_TARGET} GODOT_ARCH )

set( OUTPUT_DIR "${CMAKE_CURRENT_SOURCE_DIR}/project/bin/" )

set_target_properties( godot-cpp-test
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

    set_target_properties( godot-cpp-test
            PROPERTIES
            OUTPUT_NAME "gdexample.macos.${TEST_TARGET}"
            SUFFIX ""

            #macos options
            OSX_ARCHITECTURES "${OSX_ARCH}"
    )
endif ()
