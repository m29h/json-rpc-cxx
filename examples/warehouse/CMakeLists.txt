cmake_minimum_required(VERSION 3.14)
set(CMAKE_CXX_STANDARD 17)
project(warehouseapp VERSION 0.0.1 LANGUAGES CXX)

option(COMPILE_TESTS "Enable tests" ON)
option(COMPILE_EXAMPLES "Enable examples" ON)
option(CODE_COVERAGE "Enable coverage reporting" OFF)

include(GNUInstallDirs)
include(FetchContent)

FetchContent_Declare(json-rpc-cxx
	  GIT_REPOSITORY https://github.com/m29h/json-rpc-cxx.git
	  GIT_TAG master
)


# Warning options for the compiler
string(
  APPEND _warning_opts
  "$<$<OR:$<CXX_COMPILER_ID:Clang>,$<CXX_COMPILER_ID:AppleClang>,$<CXX_COMPILER_ID:GNU>>:-O3;-Wall;-Wextra;-std=c++11;>"
  "$<$<OR:$<CXX_COMPILER_ID:Clang>,$<CXX_COMPILER_ID:AppleClang>>:-Wthread-safety;-Wpedantic;-Weffc++;>"
  "$<$<CXX_COMPILER_ID:GNU>:-pedantic;-pedantic-errors;>"
  )

FetchContent_Declare(
  cpp-httplib
  GIT_REPOSITORY https://github.com/yhirose/cpp-httplib.git
  GIT_TAG        v0.7.5
  )
find_package(Threads)
add_executable(example-warehouse main.cpp warehouseapp.cpp types.h ../inmemoryconnector.hpp ../cpphttplibconnector.hpp)
target_compile_options(example-warehouse PUBLIC "${_warning_opts}")
FetchContent_MakeAvailable(cpp-httplib json-rpc-cxx)
target_link_libraries(example-warehouse PUBLIC json-rpc-cxx Threads::Threads httplib::httplib)
target_include_directories(example-warehouse PRIVATE ../)
add_test(NAME example COMMAND example-warehouse)

