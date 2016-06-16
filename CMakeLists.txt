# Copyright © 2013 Canonical Ltd.
#
# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License version 3 as
# published by the Free Software Foundation.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program.  If not, see <http://www.gnu.org/licenses/>.
#
# Authored by: Thomas Voss <thomas.voss@canonical.com>

cmake_minimum_required(VERSION 2.8)

project(process-cpp)

# We haven't received version information via the packaging setup.
# For that, we try to determine sensible values on our own, ensuring
# plain old invocations to cmake still work as expected.
if (NOT DEFINED PROCESS_CPP_VERSION_MAJOR)
  find_program(LSB_RELEASE lsb_release)
  execute_process(
    COMMAND ${LSB_RELEASE} -c -s
    OUTPUT_VARIABLE DISTRO_CODENAME
    OUTPUT_STRIP_TRAILING_WHITESPACE)

  # We explicitly ignore errors and only check if we are building for vivid.
  # For all other cases:
  #   - releases other than vivid
  #   - other distros
  #   - errors
  # we define the version to be 3.0.1
  if (${DISTRO_CODENAME} STREQUAL "vivid")
    set(PROCESS_CPP_VERSION_MAJOR 2)
    set(PROCESS_CPP_VERSION_MINOR 0)
    set(PROCESS_CPP_VERSION_PATCH 0)
  else ()
    set(PROCESS_CPP_VERSION_MAJOR 3)
    set(PROCESS_CPP_VERSION_MINOR 0)
    set(PROCESS_CPP_VERSION_PATCH 1)
  endif()
endif()

message(STATUS "${CMAKE_PROJECT_NAME} ${PROCESS_CPP_VERSION_MAJOR}.${PROCESS_CPP_VERSION_MINOR}.${PROCESS_CPP_VERSION_PATCH}")


find_package(Boost COMPONENTS iostreams system REQUIRED)
find_package(PkgConfig REQUIRED)
find_package(Threads REQUIRED)

pkg_check_modules(PROPERTIES_CPP properties-cpp)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

include(GNUInstallDirs)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -pedantic -Wextra -fvisibility=hidden")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -Wall -fno-strict-aliasing -fvisibility=hidden -fvisibility-inlines-hidden -pedantic -Wextra")
set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -Wl,--no-undefined")

option(PROCESS_CPP_WERROR "Treat warnings as errors" ON)

if(PROCESS_CPP_WERROR)
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Werror -Wno-error=format")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Werror -Wno-error=format")
endif(PROCESS_CPP_WERROR)

string(TOLOWER "${CMAKE_BUILD_TYPE}" cmake_build_type_lower)

#####################################################################
# Enable code coverage calculation with gcov/gcovr/lcov
# Usage:
#  * Switch build type to coverage (use ccmake or cmake-gui)
#  * Invoke make, make test, make coverage
#  * Find html report in subdir coveragereport
#  * Find xml report feasible for jenkins in coverage.xml
#####################################################################
IF(cmake_build_type_lower MATCHES coverage)
  SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -ftest-coverage -fprofile-arcs" )
  SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -ftest-coverage -fprofile-arcs" )
  SET(CMAKE_MODULE_LINKER_FLAGS "${CMAKE_MODULE_LINKER_FLAGS} -ftest-coverage -fprofile-arcs" )
  SET(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -ftest-coverage -fprofile-arcs" )
ENDIF(cmake_build_type_lower MATCHES coverage)

include(CTest)

include_directories(
  include/

  ${Boost_INCLUDE_DIRS}
  ${PROPERTIES_CPP_INCLUDE_DIRS}
)

add_subdirectory(doc)
add_subdirectory(data)
add_subdirectory(include)
add_subdirectory(src)
add_subdirectory(tests)
