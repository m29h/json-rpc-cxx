cmake_minimum_required(VERSION 3.14)
set(CMAKE_CXX_STANDARD 17)
project(json-rpc-cxx VERSION 0.0.1 LANGUAGES CXX)

option(COMPILE_TESTS "Enable tests" ON)
option(COMPILE_EXAMPLES "Enable examples" ON)
option(CODE_COVERAGE "Enable coverage reporting" OFF)

include(GNUInstallDirs)
include(FetchContent)

FetchContent_Declare(nlohmann_json
	  GIT_REPOSITORY https://github.com/ArthurSonzogni/nlohmann_json_cmake_fetchcontent
	  GIT_TAG v3.9.1
)

FetchContent_Declare(
  catch
  GIT_REPOSITORY https://github.com/catchorg/Catch2.git
  GIT_TAG        v2.13.0
)

FetchContent_Declare(
  cpp-httplib
  GIT_REPOSITORY https://github.com/yhirose/cpp-httplib.git
  GIT_TAG        v0.7.5
)



add_library(json-rpc-cxx INTERFACE)
target_include_directories(json-rpc-cxx INTERFACE include)
FetchContent_MakeAvailable(nlohmann_json)
target_link_libraries(json-rpc-cxx INTERFACE nlohmann_json::nlohmann_json )

install(DIRECTORY include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

add_library(coverage_config INTERFACE)

# Warning options for the compiler
string(
  APPEND _warning_opts
  "$<$<OR:$<CXX_COMPILER_ID:Clang>,$<CXX_COMPILER_ID:AppleClang>,$<CXX_COMPILER_ID:GNU>>:-O3;-Wall;-Wextra;-std=c++11;>"
  "$<$<OR:$<CXX_COMPILER_ID:Clang>,$<CXX_COMPILER_ID:AppleClang>>:-Wthread-safety;-Wpedantic;-Weffc++>"
  "$<$<CXX_COMPILER_ID:GNU>:-pedantic;-pedantic-errors;>"
  )


if (COMPILE_TESTS)
    if (CODE_COVERAGE)
        message("Enabled coverage flags")
        target_compile_options(coverage_config INTERFACE -O0 -g --coverage)
        target_link_libraries(coverage_config INTERFACE --coverage)
    endif ()
    add_executable(jsonrpccpp-test test/main.cpp test/client.cpp test/typemapper.cpp test/dispatcher.cpp test/server.cpp test/batchclient.cpp test/testclientconnector.hpp examples/warehouse/warehouseapp.cpp test/warehouseapp.cpp)
    target_compile_options(jsonrpccpp-test PUBLIC "${_warning_opts}")
    target_include_directories(jsonrpccpp-test PRIVATE examples)
    FetchContent_MakeAvailable(catch)
    target_link_libraries(jsonrpccpp-test coverage_config json-rpc-cxx Catch2::Catch2)
    enable_testing()
    add_test(NAME test COMMAND jsonrpccpp-test)
endif ()

if (COMPILE_EXAMPLES)
    find_package(Threads)
    add_executable(example-warehouse examples/warehouse/main.cpp examples/warehouse/warehouseapp.cpp examples/warehouse/types.h examples/inmemoryconnector.hpp)
    target_compile_options(example-warehouse PUBLIC "${_warning_opts}")
    FetchContent_MakeAvailable(cpp-httplib)
    target_link_libraries(example-warehouse PUBLIC json-rpc-cxx Threads::Threads httplib::httplib)
    #    target_include_directories(example-warehouse SYSTEM PRIVATE vendor)
    target_include_directories(example-warehouse PRIVATE examples)
    add_test(NAME example COMMAND example-warehouse)
endif ()



