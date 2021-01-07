# Copyright (c) 2015, Matthijs Möhlmann
# Copyright (c) 2016, Christian Dávid
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

set(LIB_HEADERS
    detail/prepared_statement_handle.h
)

add_library(sqlpp11-connector-postgresql
  bind_result.cpp
  connection.cpp
  exception.cpp
  prepared_statement.cpp
  detail/connection_handle.cpp
  detail/prepared_statement_handle.cpp
  result.cpp
)
if(BUILD_SHARED_LIBS)
  target_sources(sqlpp11-connector-postgresql
    detail/dynamic_libpq.cpp
  )
endif()

if (WIN32)
  get_filename_component(POSTGRESQL_LIB_FILE ${PostgreSQL_LIBRARIES} NAME)
elseif(APPLE)
  set(POSTGRESQL_LIB_FILE "lib${PostgreSQL_LIBRARIES}.dylib")
else()
  if("${PostgreSQL_LIBRARIES}" MATCHES ".+lib.+so")
	  get_filename_component(POSTGRESQL_LIB_FILE ${PostgreSQL_LIBRARIES} NAME)
  else()
      set(POSTGRESQL_LIB_FILE "${PostgreSQL_LIBRARIES}.so")
  endif()
  set(POSTGRESQL_LIB_FILE "lib${PostgreSQL_LIBRARIES}.so")
endif()
if(BUILD_SHARED_LIBS)
  target_compile_options(sqlpp11-connector-postgresql INTERFACE 
    -DSQLPP_DYNAMIC_LOADING
  )
  target_compile_options(sqlpp11-connector-postgresql PRIVATE 
    -DBUILDING_DLL 
    -DSQLPP_DYNAMIC_LOADING 
    -DSQLPP_DYNAMIC_LOADING_FILENAME=${POSTGRESQL_LIB_FILE}
  )
endif()

# Generate code coverage if enabled
if (CODE_COVERAGE AND CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
	# Add required flags for code coverage
	target_compile_options(sqlpp11-connector-postgresql PUBLIC -O0 --coverage)
	if (CMAKE_VERSION VERSION_GREATER_EQUAL 3.13)
		target_link_options(sqlpp11-connector-postgresql INTERFACE --coverage)
	else()
		target_link_libraries(sqlpp11-connector-postgresql INTERFACE --coverage)
	endif()
endif()
target_compile_features(sqlpp11-connector-postgresql PRIVATE cxx_auto_type)

target_link_libraries(sqlpp11-connector-postgresql PUBLIC sqlpp11 PostgreSQL::libpq)
message(STATUS ${CMAKE_SOURCE_DIR})
target_include_directories(sqlpp11-connector-postgresql PUBLIC
  $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
)