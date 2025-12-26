cmake_minimum_required(VERSION 3.16.0)

file(READ "${CMAKE_SOURCE_DIR}/TDSControl/VERSION" VERSION_CONTENTS) 
string(REGEX MATCH "VERSION = \"([0-9]+\.[0-9]+\.[0-9])\"" _ ${VERSION_CONTENTS})
set(PROJECT_VERSION "${CMAKE_MATCH_1}")

project(tds-control-python VERSION ${PROJECT_VERSION} LANGUAGES C CXX)

# Include the C++ library.
add_subdirectory("TDSControl" EXCLUDE_FROM_ALL)

set(TDS_CONTROL_PY_BINDINGS
        wrapper/py_bindings.cpp)

# Find python exe + nanobind cmake directory
find_package(Python 3.9 REQUIRED COMPONENTS Interpreter Development.Module REQUIRED)
execute_process(
    COMMAND "${Python_EXECUTABLE}" -m nanobind --cmake_dir
    OUTPUT_STRIP_TRAILING_WHITESPACE OUTPUT_VARIABLE nanobind_ROOT)
find_package(nanobind CONFIG REQUIRED)

nanobind_add_module(pytdscontrol MODULE ${TDS_CONTROL_PY_BINDINGS})
target_link_libraries(pytdscontrol PRIVATE tdscontrol)

# Install the compiled extension into the package dir inside the wheel
install(TARGETS pytdscontrol
        LIBRARY DESTINATION "tdscontrol"    # Unix .so
        RUNTIME DESTINATION "tdscontrol")   # Windows .pyd/.dll

