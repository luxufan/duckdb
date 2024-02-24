add_definitions(-DSQLITE_OMIT_LOAD_EXTENSION=1)

set(SHELL_SOURCES shell.c)
if(NOT WIN32)
  add_definitions(-DHAVE_LINENOISE=1)
  set(SHELL_SOURCES ${SHELL_SOURCES} linenoise.cpp)
  include_directories(../../third_party/utf8proc/include)
endif()

option(STATIC_LIBCPP "Statically link CLI to libc++" FALSE)

include_directories(include)
include_directories(../sqlite3_api_wrapper/include)
list(APPEND TEST_TARGETS duckdb_shell)
llvm_multisource(duckdb_shell ${SHELL_SOURCES})
add_dependencies(sqlite3_api_wrapper_static build-timeit)
target_link_libraries(duckdb_shell sqlite3_api_wrapper_static
                      ${DUCKDB_EXTRA_LINK_FLAGS})
link_threads(duckdb_shell)
if(STATIC_LIBCPP)
  message("Statically linking CLI")
  target_link_libraries(duckdb_shell -static-libstdc++ -static-libgcc)
endif()

if(NOT AMALGAMATION_BUILD AND NOT WIN32)
  target_link_libraries(duckdb_shell duckdb_utf8proc)
endif()

set_target_properties(duckdb_shell PROPERTIES OUTPUT_NAME duckdb)
set_target_properties(duckdb_shell PROPERTIES RUNTIME_OUTPUT_DIRECTORY
                                       ${PROJECT_BINARY_DIR})

install(TARGETS shell RUNTIME DESTINATION "${INSTALL_BIN_DIR}")
