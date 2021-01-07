# Copyright (c) 2015, Matthijs Möhlmann
# Copyright (c) 2016, Christian David
# All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions are met:
#
#   Redistributions of source code must retain the above copyright notice, this
#   list of conditions and the following disclaimer.
#
#   Redistributions in binary form must reproduce the above copyright notice,
#   this list of conditions and the following disclaimer in the documentation
#   and/or other materials provided with the distribution.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
# AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
# IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
# ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
# LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
# CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
# SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
# INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
# CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
# ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
# POSSIBILITY OF SUCH DAMAGE.

cmake_minimum_required(VERSION 3.13 FATAL_ERROR)
include(cmake/HunterGate.cmake)
set(
    HUNTER_CACHE_SERVERS
    "https://github.com/CHChang810716/hunter-binary-cache"
    CACHE
    STRING
    "Hunter cache servers"
)
HunterGate(
    URL "https://github.com/CHChang810716/hunter.git"
    SHA1 62eee92093b4cd36e80ca033689d4abb7e2def32
)
project(sqlpp11-connector-postgresql VERSION 0.54.0 LANGUAGES CXX)

option(ENABLE_TESTS "Build unit tests" ON)
option(CODE_COVERAGE "Enable coverage reporting" OFF)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

hunter_add_package(Sqlpp11)
find_package(Sqlpp11 CONFIG REQUIRED)

hunter_add_package(ZLIB)
find_package(ZLIB CONFIG REQUIRED)

hunter_add_package(PostgreSQL)
find_package(PostgreSQL REQUIRED)

hunter_add_package(Boost)
find_package(Boost CONFIG REQUIRED)

# include_directories("${PROJECT_SOURCE_DIR}/include")
# include_directories(${Boost_INCLUDE_DIRS})

add_subdirectory(src)

if (ENABLE_TESTS)
    enable_testing()
    add_subdirectory(tests)
endif()

# install(DIRECTORY "${PROJECT_SOURCE_DIR}/include/sqlpp11" DESTINATION include COMPONENT Devel)
# install(
#   FILES
#     cmake/Sqlpp-connector-postgresqlConfig.cmake
#   DESTINATION
#     ${ConfigPackageLocation}
#   COMPONENT
#     Devel
# )

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)
set(AKT_GEN_DIR             "${CMAKE_CURRENT_BINARY_DIR}/generated")
set(AKT_CONFIG_INSTALL_DIR  "lib/cmake/${PROJECT_NAME}")
set(AKT_CONFIG_VERSION      "${AKT_GEN_DIR}/${PROJECT_NAME}ConfigVersion.cmake")
set(AKT_PROJECT_CONFIG      "${AKT_GEN_DIR}/${PROJECT_NAME}Config.cmake")
set(AKT_TARGETS_EXPORT      "${PROJECT_NAME}Targets")
set(AKT_NAMESPACE           "${PROJECT_NAME}::")
configure_package_config_file(
    cmake/Config.cmake.in
    ${AKT_PROJECT_CONFIG}
    INSTALL_DESTINATION ${AKT_CONFIG_INSTALL_DIR}
)
write_basic_package_version_file(
    ${AKT_CONFIG_VERSION} COMPATIBILITY SameMajorVersion
)
install(
    FILES ${AKT_PROJECT_CONFIG}
    DESTINATION ${AKT_CONFIG_INSTALL_DIR}
)
install(
    TARGETS sqlpp11-connector-postgresql
    EXPORT ${AKT_TARGETS_EXPORT}
    BUNDLE DESTINATION . COMPONENT Runtime
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT Runtime
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT Library
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT Develop
)
install(
    EXPORT ${AKT_TARGETS_EXPORT}
    NAMESPACE ${AKT_NAMESPACE}
    DESTINATION ${AKT_CONFIG_INSTALL_DIR}
)
