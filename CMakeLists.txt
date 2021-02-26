cmake_minimum_required(VERSION 3.13)

project(graphql-parser-base VERSION 0.1.0)

if (NOT EXISTS ${CMAKE_BINARY_DIR}/CMakeCache.txt)
    if (NOT CMAKE_BUILD_TYPE)
        set(CMAKE_BUILD_TYPE "Release" CACHE STRING "" FORCE)
    endif ()
endif ()

set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -Wall -Wextra -g -O0")

if (DEFINED ${GRAPHQL_BASE_PARSER_MARCH})
    set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -march=${GRAPHQL_BASE_PARSER_MARCH}")
endif ()

# build antlr4
LIST(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/antlr4cmake)

set(ANTLR4_WITH_STATIC_CRT OFF)
add_definitions(-DANTLR4CPP_STATIC)

if (CMAKE_SYSTEM_NAME MATCHES "Linux")
    find_package(PkgConfig REQUIRED)
    pkg_check_modules(UUID REQUIRED uuid)
endif ()

find_package(Java COMPONENTS Runtime REQUIRED)

set(ANTLR4_TAG "4.8")
set(ANTLR_EXECUTABLE "${PROJECT_BINARY_DIR}/antlr-${ANTLR4_TAG}-complete.jar")
if (NOT EXISTS "${ANTLR_EXECUTABLE}")
    file(DOWNLOAD https://www.antlr.org/download/antlr-${ANTLR4_TAG}-complete.jar ${ANTLR_EXECUTABLE} SHOW_PROGRESS)
endif ()
SET(ANTLR4_INSTALL ON)
include(ExternalAntlr4Cpp)

# add antrl4cpp artifacts to project environment
include_directories(${ANTLR4_INCLUDE_DIRS})
#file(CREATE_LINK ${CMAKE_CURRENT_SOURCE_DIR}/GraphQL.g4
#        ${CMAKE_CURRENT_SOURCE_DIR}/GraphQL.g4
#        )

# add macros to generate ANTLR Cpp code from grammar
find_package(ANTLR REQUIRED)

message("Generating GraphQL Parser.")
antlr_target(GraphQL GraphQL.g4 LEXER PARSER LISTENER VISITOR
        PACKAGE Dice::graphQL_parser::base
        )

# include generated files in project environment
add_library(graphql-parser-base STATIC
        ${ANTLR_GraphQL_CXX_OUTPUTS})
target_include_directories(graphql-parser-base PUBLIC
        ${ANTLR_GRAPHQL_OUTPUT_DIR})

set_target_properties(graphql-parser-base PROPERTIES
        CXX_STANDARD 17)

target_link_libraries(graphql-parser-base PUBLIC antlr4_static)


add_dependencies(graphql-parser-base
        antlr4_static)

target_include_directories(graphql-parser-base PUBLIC
        ${ANTLR4_INCLUDE_DIRS}
        ${PROJECT_SOURCE_DIR}/generated
        )
add_library(graphql-parser-base::graphql-parser-base ALIAS graphql-parser-base)

# Make package findable
configure_file(cmake/dummy-config.cmake.in graphql-parser-base-config.cmake @ONLY)
configure_file(antlr4cmake/antlr4-runtime.cmake.in antlr4-runtime-config.cmake @ONLY)

# Enable version checks in find_package
include(CMakePackageConfigHelpers)
write_basic_package_version_file(graphql-parser-base-config-version.cmake COMPATIBILITY SameMajorVersion)
write_basic_package_version_file(antlr4-runtime-config-version.cmake COMPATIBILITY SameMajorVersion)

# install and export target
install(TARGETS graphql-parser-base
        EXPORT graphql-parser-base-targets
        ARCHIVE DESTINATION lib)


    install(DIRECTORY "${ANTLR_GRAPHQL_OUTPUT_DIR}"
            DESTINATION "include"
            FILES_MATCHING PATTERN "*.h"
            )

install(DIRECTORY
        "${ANTLR4_INCLUDE_DIRS}/"
        DESTINATION "include"
        FILES_MATCHING PATTERN "*.h"
        )

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/antlr4-runtime-config-version.cmake DESTINATION "share/antlr4-runtime/cmake")

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/graphql-parser-base-config-version.cmake DESTINATION "share/graphql-parser-base/cmake")

if (EXISTS LICENSE.txt)
    install(FILES LICENSE.txt
            DESTINATION "share/doc/graphql-parser-base")
elseif (EXISTS ../../LICENSE.txt)
    install(FILES ../../LICENSE.txt
            DESTINATION "share/doc/graphql-parser-base")
endif ()

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/antlr4_runtime/src/antlr4_runtime/LICENSE.txt
        DESTINATION "share/licenses/antlr4-runtime")
