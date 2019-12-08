# Copyright 2018-2019 Mike Dev
# Distributed under the Boost Software License, Version 1.0.
# See accompanying file LICENSE_1_0.txt or copy at https://www.boost.org/LICENSE_1_0.txt

cmake_minimum_required( VERSION 3.5 )
# NOTE: Individual boost cmake files might require a higher cmake version
if( POLICY CMP0074 )
  cmake_policy( SET CMP0074 NEW )
endif()
if( POLICY CMP0069 )
  cmake_policy( SET CMP0069 NEW )
endif()

project( BoostMinCMake LANGUAGES CXX )

message( "
================================ BOOST WARNING ===================================================
=                                                                                                =
= General cmake support in boost is EXPERIMENTAL at best and                                     =
= only a few libraries can be build via cmake at all.                                            =
=                                                                                                =
= The officially supported way to build and test boost is using b2, as e.g. explained here:      =
= https://www.boost.org/doc/libs/1_68_0/more/getting_started/index.html                          =
=                                                                                                =
==================================================================================================
")

# ======= options ==========
option( BOOST_MIN_CMAKE_INCLUDE_ALL_TESTS "Include boost tests (where known to cmake)" ON)

# exclude libraries that don't work with the superproject yet
# (e.g. due to missing dependencies or conflicting target definitions)
set( BOOST_MIN_CMAKE_IGNORE_LIBS beast;callable_traits;compute;gil;hana;safe_numerics;serialization;yap
     CACHE STRING "List of libraries that will be excluded from cmake build (format: lib1;lib2;lib3)" )

# Root of boost distribution we want to compile.
# This file is supposed to live in the root of that directory,
# so CMAKE_CURRENT_SOURCE_DIR} is the default
set( BOOST_MIN_CMAKE_BOOST_ROOT "${CMAKE_CURRENT_SOURCE_DIR}"
     CACHE STRING "Root of boost that is supposed to be build with cmake" )

# ~~~~~~~ options ~~~~~~~~~~

message( STATUS "[${CMAKE_PROJECT_NAME}] Manually excluded libs: ${BOOST_MIN_CMAKE_IGNORE_LIBS}" )

# Override usual boost_root environment variable to prevent inconsistencies
set( ENV{BOOST_ROOT} "${BOOST_MIN_CMAKE_BOOST_ROOT}" )



# cmake doesn't require autolinking and currently most cmake files don't produce
# name mangled libraries anyway
message( STATUS "[${CMAKE_PROJECT_NAME}] Deactivating boost auto linking mechanism (-DBOOST_ALL_NO_LIB)" )
add_definitions( -DBOOST_ALL_NO_LIB )


# Detect all CMakeLists files that reside in the root folder of a library
file( GLOB boost_libs_with_cmake_files ${BOOST_MIN_CMAKE_BOOST_ROOT}/libs/*/CMakeLists.txt )
file( GLOB boost_numeric_libs_with_cmake_files ${BOOST_MIN_CMAKE_BOOST_ROOT}/libs/numeric/*/CMakeLists.txt )
list( APPEND boost_libs_with_cmake_files ${boost_numeric_libs_with_cmake_files} )

set( tmp "${boost_libs_with_cmake_files}")
list(TRANSFORM tmp REPLACE "${BOOST_MIN_CMAKE_BOOST_ROOT}/libs/" "")
list(TRANSFORM tmp REPLACE  "/CMakeLists\.txt" "")
list(TRANSFORM tmp REPLACE  "/" "_")
list(REMOVE_ITEM tmp ${BOOST_MIN_CMAKE_IGNORE_LIBS})
message( STATUS "[${CMAKE_PROJECT_NAME}] Adding following boost libraries: ${tmp}" )


enable_testing()
foreach( cmake_file IN LISTS boost_libs_with_cmake_files )

    get_filename_component( dir ${cmake_file} DIRECTORY )

	file(RELATIVE_PATH rel ${BOOST_MIN_CMAKE_BOOST_ROOT}/libs ${dir})

	string(REPLACE "/" "_" lib_name ${rel})
	string(TOUPPER ${lib_name} lib_macro_prefix )

    if(NOT lib_name IN_LIST BOOST_MIN_CMAKE_IGNORE_LIBS )
        message( VERBOSE "[${CMAKE_PROJECT_NAME}] Adding library ${lib_name}" )

		if(BOOST_MIN_CMAKE_INCLUDE_ALL_TESTS)
			set(BOOST_${lib_macro_prefix}_INCLUDE_TESTS ON CACHE BOOL "")
		endif()

		add_subdirectory( ${dir} ${CMAKE_CURRENT_BINARY_DIR}/${lib_name})

    endif()

endforeach()
