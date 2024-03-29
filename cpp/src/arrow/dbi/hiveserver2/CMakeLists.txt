# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

add_custom_target(arrow_hiveserver2)
add_custom_target(arrow_hiveserver2-tests)

# Headers: top level
arrow_install_all_headers("arrow/dbi/hiveserver2")

set(ARROW_HIVESERVER2_SRCS
    columnar_row_set.cc
    service.cc
    session.cc
    operation.cc
    sample_usage.cc
    thrift_internal.cc
    types.cc
    util.cc)

set(HIVESERVER2_THRIFT_SRC_DIR "${ARROW_BINARY_DIR}/src/arrow/dbi/hiveserver2")
file(MAKE_DIRECTORY ${HIVESERVER2_THRIFT_SRC_DIR})
add_subdirectory(thrift)

# *_constants.* aren't generated when "const" doesn't exist in *.thrift.
set(HIVESERVER2_THRIFT_SRC
    ${HIVESERVER2_THRIFT_SRC_DIR}/ErrorCodes_constants.cpp
    ${HIVESERVER2_THRIFT_SRC_DIR}/ErrorCodes_types.cpp
    ${HIVESERVER2_THRIFT_SRC_DIR}/ImpalaService.cpp
    # ${HIVESERVER2_THRIFT_SRC_DIR}/ImpalaService_constants.cpp
    ${HIVESERVER2_THRIFT_SRC_DIR}/ImpalaService_types.cpp
    ${HIVESERVER2_THRIFT_SRC_DIR}/ImpalaHiveServer2Service.cpp
    # ${HIVESERVER2_THRIFT_SRC_DIR}/beeswax_constants.cpp
    ${HIVESERVER2_THRIFT_SRC_DIR}/beeswax_types.cpp
    ${HIVESERVER2_THRIFT_SRC_DIR}/BeeswaxService.cpp
    ${HIVESERVER2_THRIFT_SRC_DIR}/TCLIService.cpp
    ${HIVESERVER2_THRIFT_SRC_DIR}/TCLIService_constants.cpp
    ${HIVESERVER2_THRIFT_SRC_DIR}/TCLIService_types.cpp
    # ${HIVESERVER2_THRIFT_SRC_DIR}/ExecStats_constants.cpp
    ${HIVESERVER2_THRIFT_SRC_DIR}/ExecStats_types.cpp
    ${HIVESERVER2_THRIFT_SRC_DIR}/hive_metastore_constants.cpp
    ${HIVESERVER2_THRIFT_SRC_DIR}/hive_metastore_types.cpp
    # ${HIVESERVER2_THRIFT_SRC_DIR}/Status_constants.cpp
    ${HIVESERVER2_THRIFT_SRC_DIR}/Status_types.cpp
    # ${HIVESERVER2_THRIFT_SRC_DIR}/Types_constants.cpp
    ${HIVESERVER2_THRIFT_SRC_DIR}/Types_types.cpp)

set_source_files_properties(${HIVESERVER2_THRIFT_SRC}
                            PROPERTIES COMPILE_FLAGS
                                       "-Wno-unused-variable -Wno-shadow-field" GENERATED
                                                                                TRUE)

# keep everything in one library, the object files reference
# each other
add_library(arrow_hiveserver2_thrift STATIC ${HIVESERVER2_THRIFT_SRC})

# Setting these files as code-generated lets make clean and incremental builds work
# correctly

# TODO(wesm): Something is broken with the dependency chain with
# ImpalaService.cpp and others. Couldn't figure out what is different between
# this setup and Impala.

add_dependencies(arrow_hiveserver2_thrift hs2-thrift-cpp)

set_target_properties(arrow_hiveserver2_thrift
                      PROPERTIES LIBRARY_OUTPUT_DIRECTORY
                                 "${BUILD_OUTPUT_ROOT_DIRECTORY}")

add_arrow_lib(arrow_hiveserver2
              SOURCES
              ${ARROW_HIVESERVER2_SRCS}
              OUTPUTS
              ARROW_HIVESERVER2_LIBRARIES
              DEPENDENCIES
              arrow_hiveserver2_thrift
              SHARED_LINK_FLAGS
              ""
              SHARED_LINK_LIBS
              ${ARROW_PYTHON_SHARED_LINK_LIBS})

add_dependencies(arrow_hiveserver2 ${ARROW_HIVESERVER2_LIBRARIES})

foreach(LIB_TARGET ${ARROW_HIVESERVER2_LIBRARIES})
  target_compile_definitions(${LIB_TARGET} PRIVATE ARROW_EXPORTING)
endforeach()

set_property(SOURCE ${ARROW_HIVESERVER2_SRCS}
             APPEND_STRING
             PROPERTY COMPILE_FLAGS " -Wno-shadow-field")

set(ARROW_HIVESERVER2_TEST_LINK_LIBS arrow_hiveserver2_static arrow_hiveserver2_thrift
                                     ${ARROW_TEST_LINK_LIBS} thrift::thrift)

if(ARROW_BUILD_TESTS)
  add_test_case(hiveserver2_test
                STATIC_LINK_LIBS
                "${ARROW_HIVESERVER2_TEST_LINK_LIBS}"
                LABELS
                "arrow_hiveserver2-tests")
  if(TARGET arrow-hiveserver2-test)
    set_property(TARGET arrow-hiveserver2-test
                 APPEND_STRING
                 PROPERTY COMPILE_FLAGS " -Wno-shadow-field")
  endif()
endif(ARROW_BUILD_TESTS)
