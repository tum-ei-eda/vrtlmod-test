####################################################################################################
# Copyright 2022 Chair of EDA, Technical University of Munich
#
# Licensed under the Apache License, Version 2.0 (the License);
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an AS IS BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
####################################################################################################

cmake_minimum_required(VERSION 3.15 FATAL_ERROR)
list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)
project(vrtlmod-test)

if(NOT CMAKE_CXX_STANDARD)
    set(CMAKE_CXX_STANDARD 17)
    message(WARNING "CMAKE_CXX_STANDARD not set. Default to ${CMAKE_CXX_STANDARD}")
endif()

set(VRTL_NAME ${VP}) #set(TOP_NAME ...) #from PARENT_SCOPE

if(NOT VERILATOR_ROOT)
    set(VERILATOR_ROOT $ENV{VERILATOR_ROOT})
    message(STATUS "VERILATOR_ROOT not specified in CMake .. using environment VERILATOR_ROOT: $ENV{VERILATOR_ROOT}")
endif()
find_package(VERILATOR REQUIRED)
if(NOT VRTLMOD_ROOT)
    set(VRTLMOD_ROOT $ENV{VRTLMOD_ROOT})
    message(STATUS "VRTLMOD_ROOT not specified in CMake .. using environment VRTLMOD_ROOT: $ENV{VRTLMOD_ROOT}")
endif()
if(NOT CMAKE_CXX_STANDARD)
    set(CMAKE_CXX_STANDARD 17)
    message(WARNING "CMAKE_CXX_STANDARD not set. Default to ${CMAKE_CXX_STANDARD}")
endif()

#list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_BINARY_DIR})
#list(APPEND CMAKE_PREFIX_PATH ${CMAKE_CURRENT_BINARY_DIR})

if(NOT SystemCLanguage_DIR)
    message(STATUS "SystemCLanguage_DIR not specified in CMake .. trying environment SYSTEMC_HOME: ${SYSTEMC_HOME}")
    if(NOT SYSTEMC_HOME)
        message(STATUS "SYSTEMC_HOME not specified in CMake .. trying environment SYSTEMC_HOME: $ENV{SYSTEMC_HOME}")
        set(SYSTEMC_HOME "$ENV{SYSTEMC_HOME}")
    endif()
    set(SystemCLanguage_DIR "${SYSTEMC_HOME}/lib/cmake/SystemCLanguage")
endif()
find_package(SystemCLanguage REQUIRED)
get_target_property(SYSTEMC_INCLUDE_DIRS SystemC::systemc INTERFACE_INCLUDE_DIRECTORIES)
set(SystemC_LIBRARIES SystemC::systemc)

####################################################################################################
# third party sources:
add_subdirectory(third_party)

add_subdirectory(benchmark)