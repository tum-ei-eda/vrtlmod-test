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
project(vrtlmod-benchmark)

if(NOT CMAKE_CXX_STANDARD)
    set(CMAKE_CXX_STANDARD 17)
    message(WARNING "CMAKE_CXX_STANDARD not set. Default to ${CMAKE_CXX_STANDARD}")
endif()
if(NOT VRTL_NAMES)
  set(VRTL_NAMES "foo" "cv32e40p" "cv32e40s" "cva6")
endif()

include(FetchContent)

####################################################################################################
# rtl sources: set RTL_SOURCES, RTL_INCLUDE_DIRS, and VERILATE_ARGS in PARENT_SCOPE:
foreach(VRTL_NAME IN ITEMS "${VRTL_NAMES}")
    message("VRTL_NAME: ${VRTL_NAME}")
    add_subdirectory(${VRTL_NAME})
    message(${RTL_SRCS})

    set(TOP_NAME ${TOP_NAME})

    file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/null.cpp "")

    add_library(V${VRTL_NAME} OBJECT
        ${CMAKE_CURRENT_BINARY_DIR}/null.cpp
    )

    verilate(V${VRTL_NAME}
        PREFIX V${TOP_NAME}
        TOP_MODULE ${TOP_NAME}
        SYSTEMC
        INCLUDE_DIRS ${RTL_INCLUDE_DIRS}
        DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/${VRTL_NAME}/obj_dir
        VERILATOR_ARGS ${VERILATE_ARGS} +define+VRTLFI
        SOURCES ${RTL_SRCS}
    )

    target_link_libraries(V${VRTL_NAME} PUBLIC
        ${SystemC_LIBRARIES}
    )
    target_include_directories(V${VRTL_NAME} PUBLIC
        ${VERILATOR_INCLUDE_DIRECTORY}
        ${VERILATOR_INCLUDE_DIRECTORY}/vltstd
        ${CMAKE_CURRENT_BINARY_DIR}/${VRTL_NAME}/obj_dir
    )

    get_target_property(SYSTEMC_INCLUDE_DIRS SystemC::systemc INTERFACE_INCLUDE_DIRECTORIES)

    file(GLOB CC_SOURCES ${CMAKE_CURRENT_BINARY_DIR}/${VRTL_NAME}/obj_dir/*)

    find_package(vrtlmod HINTS $ENV{VRTLMOD_ROOT} ${VRTLMOD_ROOT} REQUIRED)

    set(VRTL_DIR ${VRTL_NAME}/obj_dir)

    file( GLOB VRTL_SRCS ${CMAKE_CURRENT_BINARY_DIR}/${VRTL_DIR}/*.cpp)

    set(VRTLMOD_OUTSOURCES ${VRTL_SRCS})

    set(VRTLMOD_OUT_DIR "${VRTL_DIR}/vrtlmod/")

    list(TRANSFORM VRTLMOD_OUTSOURCES REPLACE "/${VRTL_DIR}/" "/${VRTLMOD_OUT_DIR}/")

    message("vrtlmod outsources" ${VRTLMOD_OUTSOURCES})

    if(NOT EXISTS "${CMAKE_CURRENT_BINARY_DIR}/${VRTLMOD_OUT_DIR}")
        set(MAKE_VRTLMOD On)
    else()
        set(VARSET ${VRTLMOD_OUTSOURCES} ${CMAKE_CURRENT_BINARY_DIR}/${VRTLMOD_OUT_DIR}/V${VRTL_NAME}_vrtlmodapi.cpp)
        foreach(var ${VARSET})
            if( NOT EXISTS "${var}" )
                set(MAKE_VRTLMOD On)
            endif()
        endforeach()
    endif()

    message("run vrtlmod: " ${MAKE_VRTLMOD})

    set(GENERATED_API_SRCS
        ${CMAKE_CURRENT_BINARY_DIR}/${VRTLMOD_OUT_DIR}/V${VRTL_NAME}_vrtlmodapi.cpp
        ${CMAKE_CURRENT_BINARY_DIR}/${VRTLMOD_OUT_DIR}/V${VRTL_NAME}_vrtlmod_diffapi.cpp
        ${CMAKE_CURRENT_BINARY_DIR}/${VRTLMOD_OUT_DIR}/V${VRTL_NAME}_vrtlmod_diffapi_compare_fast.cpp
        ${CMAKE_CURRENT_BINARY_DIR}/${VRTLMOD_OUT_DIR}/V${VRTL_NAME}_vrtlmod_diffapi_compare.cpp
        ${CMAKE_CURRENT_BINARY_DIR}/${VRTLMOD_OUT_DIR}/V${VRTL_NAME}_vrtlmod_diffapi_compute.cpp
    )


    if(MAKE_VRTLMOD)
        vrtlmod(
            SOURCES ${CC_SOURCES}
            OUTPUT ${VRTLMOD_OUTSOURCES} ${GENERATED_API_SRCS}
    #        VERBOSE
            SYSTEMC
            OUT_DIR ${CMAKE_CURRENT_BINARY_DIR}/${VRTLMOD_OUT_DIR}
            WHITELIST_XML ${WHITELIST_XML}
            INCLUDE_DIRS ${CMAKE_CURRENT_BINARY_DIR}/${VRTL_DIR}/ ${VERILATOR_INCLUDE_DIRECTORY} ${VERILATOR_INCLUDE_DIRECTORY}/vltstd
            SYSTEMC_INCLUDE_DIRS ${SYSTEMC_INCLUDE_DIRS}
        )
    endif()

    add_library(V${VRTL_NAME}_vrtlmod OBJECT
        ${CMAKE_CURRENT_BINARY_DIR}/null.cpp
        ${VRTLMOD_OUTSOURCES}
        ${VERILATOR_INCLUDE_DIRECTORY}/verilated.cpp
        ${VERILATOR_INCLUDE_DIRECTORY}/verilated_dpi.cpp
        ${GENERATED_API_SRCS}
    )
    target_link_libraries(V${VRTL_NAME}_vrtlmod PUBLIC
        ${SystemC_LIBRARIES}
    )
    target_include_directories(V${VRTL_NAME}_vrtlmod PUBLIC
        ${VERILATOR_INCLUDE_DIRECTORY}
        ${VERILATOR_INCLUDE_DIRECTORY}/vltstd
        ${CMAKE_CURRENT_BINARY_DIR}/${VRTLMOD_OUT_DIR}
    )

    message("VRTL_SRCS after vrtlmod: ${VRTLMOD_OUTSOURCES}")

    add_executable(V${VRTL_NAME}-SC_TEST
        ${VRTL_NAME}/sc_test.cpp
    )
    target_link_libraries(V${VRTL_NAME}-SC_TEST
        V${VRTL_NAME}
        scc
        elfio::elfio
        ${TGT_SPECIFIC_LIBS}
    )
    target_include_directories(V${VRTL_NAME}-SC_TEST PUBLIC
        common
    )
    add_executable(V${VRTL_NAME}-SC_TEST_VRTLMOD
        ${VRTL_NAME}/sc_test.cpp
    )
    target_compile_options(V${VRTL_NAME}-SC_TEST_VRTLMOD PUBLIC -DVRTLMOD)
    target_link_libraries(V${VRTL_NAME}-SC_TEST_VRTLMOD
        V${VRTL_NAME}_vrtlmod
        scc
        elfio::elfio
        ${TGT_SPECIFIC_LIBS}
    )
    target_include_directories(V${VRTL_NAME}-SC_TEST_VRTLMOD PUBLIC
        common
    )
endforeach()