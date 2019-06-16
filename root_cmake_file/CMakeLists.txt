# Copyright 2018-2019 Mike Dev
# Distributed under the Boost Software License, Version 1.0.
# See accompanying file LICENSE_1_0.txt or copy at https://www.boost.org/LICENSE_1_0.txt

cmake_minimum_required( VERSION 3.5 )
# NOTE: Individual boost cmake files might require a higher cmake version

if( POLICY CMP0074 )
  cmake_policy( SET CMP0074 NEW )
endif()

project( boost LANGUAGES CXX )
set( CMAKE_CXX_STANDARD 17 )

#=== options ===

# exclude libraries that don't work with the superproject yet
# (e.g. due to missing dependencies or conflicting target definitions)

set( BOOST_MAIN_IGNORE_LIBS beast;callable_traits;compute;gil;hana;hof;safe_numerics;serialization;yap CACHE STRING "List of libraries that will be excluded from cmake build (format: lib1;lib2;lib3)" )

#~~~ options ~~~


message( "
================================ BOOST WARNING ===================================================
=                                                                                                =
= General cmake support in boost is EXPERIMENTAL at best and                                     =
= only a few libraries can be build via cmake at all.                                            =
=                                                                                                =
= The officially supported way to build and test boost is still b2, as e.g. explained here:      =
= https://www.boost.org/doc/libs/1_68_0/more/getting_started/index.html                          =
=                                                                                                =
==================================================================================================
")

message( STATUS "Manually excluded libs: ${BOOST_MAIN_IGNORE_LIBS}" )

# cmake doesn't require autolinking and currently most cmake files don't produce
# name mangled libraries anyway
message( STATUS "Deactivating boost auto linking mechanism (-DBOOST_ALL_NO_LIB)" )
add_definitions( -DBOOST_ALL_NO_LIB )

# Detect and process all CMakeLists files that reside in the root folder of a library
file( GLOB boost_libs_with_cmake_files libs/*/CMakeLists.txt )
file( GLOB boost_numeric_libs_with_cmake_files libs/numeric/*/CMakeLists.txt )
set(boost_libs_with_cmake_files ${boost_libs_with_cmake_files};${boost_numeric_libs_with_cmake_files})

enable_testing()
foreach( cmake_file IN LISTS boost_libs_with_cmake_files )

    get_filename_component( dir ${cmake_file} DIRECTORY )
    get_filename_component( lib_name ${dir} NAME )

    if(NOT lib_name IN_LIST BOOST_MAIN_IGNORE_LIBS )
        message( STATUS "Adding library ${dir}" )
        add_subdirectory( ${dir} )
    endif()

endforeach()
