#
#
#  Copyright 2016 CUBRID Corporation
# 
#   Licensed under the Apache License, Version 2.0 (the "License");
#   you may not use this file except in compliance with the License.
#   You may obtain a copy of the License at
# 
#       http://www.apache.org/licenses/LICENSE-2.0
# 
#   Unless required by applicable law or agreed to in writing, software
#   distributed under the License is distributed on an "AS IS" BASIS,
#   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#   See the License for the specific language governing permissions and
#   limitations under the License.
# 
#

cmake_minimum_required(VERSION 2.8)

project(CUBRID-JDBC)

message(STATUS "=========== JDBC ===========")

# check environment
include(FindJava)
find_package(Java 1.8 COMPONENTS Development)

find_program(ANT ant PATHS $ENV{ANT_HOME} ENV PATH)
if(NOT ANT)
  message(FATAL_ERROR "The ANT was not found. check PATH or install ant to build JDBC")
endif(NOT ANT)

# check built as sub_directory
if (EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/CMakeLists.txt)
  set (CUBRID_JDBC_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR})
else (EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/CMakeLists.txt)
  # version may exist at the parent path
  set (CUBRID_JDBC_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../)
endif (EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/CMakeLists.txt)

set (CUBRID_JDBC_OUTPUT_DIR ${CUBRID_JDBC_SOURCE_DIR})
set (VERSION_FILE ${CUBRID_JDBC_OUTPUT_DIR}/VERSION-DIST)
set (JDBC_JAR ${CUBRID_JDBC_OUTPUT_DIR}/cubrid_jdbc.jar)

message(STATUS "[] CUBRID_JDBC_SOURCE_DIR = ${CUBRID_JDBC_SOURCE_DIR}")
message(STATUS "[INFO] CUBRID_JDBC_OUTPUT_DIR = ${CUBRID_JDBC_OUTPUT_DIR}")

# Generate version file
add_custom_command(
  OUTPUT ${VERSION_FILE}
  COMMAND ${CMAKE_COMMAND} -E make_directory ${CUBRID_JDBC_OUTPUT_DIR}
  COMMAND ${CMAKE_COMMAND} 
          -DCUBRID_JDBC_SOURCE_DIR=${CUBRID_JDBC_SOURCE_DIR}
          -DCUBRID_JDBC_OUTPUT_DIR=${CUBRID_JDBC_OUTPUT_DIR}
          -P ${CUBRID_JDBC_SOURCE_DIR}/cmake/gen_version.cmake
  DEPENDS ${CUBRID_JDBC_SOURCE_DIR}/build.xml
  COMMENT "Generating JDBC VERSION-DIST file..."
)

# Build JDBC driver
add_custom_command(
  OUTPUT ${JDBC_JAR}
  COMMAND ${ANT} dist-cubrid -buildfile ${CMAKE_CURRENT_SOURCE_DIR}/build.xml
  DEPENDS ${VERSION_FILE}
          ${CMAKE_CURRENT_SOURCE_DIR}/build.xml
          ${CMAKE_CURRENT_SOURCE_DIR}/src
  COMMENT "Building JDBC driver with Ant..."
)

# Create symlink
add_custom_command(
  OUTPUT ${CUBRID_JDBC_OUTPUT_DIR}/cubrid_jdbc_symlink
  COMMAND ${CMAKE_COMMAND}
          -DCUBRID_JDBC_SOURCE_DIR=${CUBRID_JDBC_SOURCE_DIR}
          -DCUBRID_JDBC_OUTPUT_DIR=${CUBRID_JDBC_OUTPUT_DIR}
          -P ${CUBRID_JDBC_SOURCE_DIR}/cmake/gen_symlink.cmake
  COMMAND ${CMAKE_COMMAND} -E touch ${CUBRID_JDBC_OUTPUT_DIR}/cubrid_jdbc_symlink
  DEPENDS ${JDBC_JAR}
  COMMENT "Generating JDBC symlink..."
)

# Main target that depends on all outputs
add_custom_target(jdbc_build ALL
  DEPENDS ${VERSION_FILE}
          ${JDBC_JAR}
          ${CUBRID_JDBC_OUTPUT_DIR}/cubrid_jdbc_symlink
)

# Install rules if needed
install(FILES ${JDBC_JAR} DESTINATION jdbc)
