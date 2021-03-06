# @@@LICENSE
#
#      Copyright (c) 2011-2013 LG Electronics, Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
# LICENSE@@@

#
# PmLogDaemon/CMakeLists.txt
#

cmake_minimum_required(VERSION 2.8.7)

project(PmLogDaemon C)

include(webOS/webOS)
webos_modules_init(1 0 0 QUALIFIER RC4)
webos_component(3 0 0)

include(FindPkgConfig)

# Check that we have glib-2.0
pkg_check_modules(GLIB2 REQUIRED glib-2.0)
include_directories(${GLIB2_INCLUDE_DIRS})
webos_add_compiler_flags(ALL ${GLIB2_CFLAGS_OTHER})

# Check that we have PmLogLib
pkg_check_modules(PMLOGLIB REQUIRED PmLogLib)
include_directories(${PMLOGLIB_INCLUDE_DIRS})
webos_add_compiler_flags(ALL ${PMLOGLIB_CFLAGS_OTHER})

# Check that we have PmLogLib-private
pkg_check_modules(PMLOGLIB_PRV REQUIRED PmLogLib-private)
include_directories(${PMLOGLIB_PRV_INCLUDE_DIRS})
webos_add_compiler_flags(ALL ${PMLOGLIB_PRV_CFLAGS_OTHER})

# Check that we have librdx
pkg_check_modules(LIBRDX REQUIRED rdx)
include_directories(${LIBRDX_INCLUDE_DIRS})
webos_add_compiler_flags(ALL ${LIBRDX_CFLAGS_OTHER})

# Check that we have libpbnjson
pkg_check_modules(PBNJSON_C REQUIRED pbnjson_c)
include_directories(${PBNJSON_C_INCLUDE_DIRS})
webos_add_compiler_flags(ALL ${PBNJSON_C_CFLAGS_OTHER})

# Check that we have luna-service2
pkg_check_modules(LUNASERVICE2 REQUIRED luna-service2)
include_directories(${LUNASERVICE2_INCLUDE_DIRS})
webos_add_compiler_flags(ALL ${LUNASERVICE2_CFLAGS_OTHER})

# Get the zlib library
include(FindZLIB)
if(NOT ZLIB_FOUND)
        message(FATAL_ERROR "ZLIB not found!")
endif()

include_directories(src)
webos_configure_header_files(src)
webos_add_compiler_flags(ALL -Wall -Wconversion)

file(GLOB SOURCE_FILES src/*.c)
webos_configure_source_files(SOURCE_FILES src/main.c)

# Build the PmLogDaemon executable
add_executable(PmLogDaemon ${SOURCE_FILES})
target_link_libraries(PmLogDaemon
                     ${GLIB2_LDFLAGS}
                     ${PMLOGLIB_LDFLAGS}
                     ${PMLOGLIB_PRV_LDFLAGS}
                     ${LIBRDX_LDFLAGS}
                     ${PBNJSON_C_LDFLAGS}
                     ${ZLIB_LIBRARIES}
                     ${LUNASERVICE2_LDFLAGS}
                     -lrt)

webos_build_daemon()
webos_build_system_bus_files()
install(PROGRAMS scripts/public/show_disk_usage.sh DESTINATION @WEBOS_INSTALL_DATADIR@/PmLogDaemon)

webos_config_build_doxygen(doc Doxyfile)
