cmake_minimum_required(VERSION 3.15)


project(LLVM LANGUAGES C CXX ASM)

set(CMAKE_CXX_STANDARD
    14
    CACHE STRING "C++ standard to conform to")
set(CMAKE_CXX_STANDARD_REQUIRED YES)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_SOURCE_DIR}/cmake"
                      "${CMAKE_CURRENT_SOURCE_DIR}/cmake/modules")


set(LLVM_TOOLS_BINARY_DIR ${LLVM_RUNTIME_OUTPUT_INTDIR}) # --bindir
set(LLVM_LIBRARY_DIR ${LLVM_LIBRARY_OUTPUT_INTDIR}) # --libdir
set(LLVM_MAIN_SRC_DIR ${CMAKE_CURRENT_SOURCE_DIR}) # --src-root
set(LLVM_MAIN_INCLUDE_DIR ${LLVM_MAIN_SRC_DIR}/include CACHE STRING "" FORCE) # --includedir
set(LLVM_BINARY_DIR ${CMAKE_CURRENT_BINARY_DIR}) # --prefix

# Note: LLVM_CMAKE_PATH does not include generated files
set(LLVM_CMAKE_PATH ${LLVM_MAIN_SRC_DIR}/cmake/modules)
set(LLVM_EXAMPLES_BINARY_DIR ${LLVM_BINARY_DIR}/examples)
set(LLVM_INCLUDE_DIR ${CMAKE_CURRENT_BINARY_DIR}/include)


option(LLVM_OPTIMIZED_TABLEGEN "Force TableGen to be built with optimization" OFF)
if(CMAKE_CROSSCOMPILING OR (LLVM_OPTIMIZED_TABLEGEN AND (LLVM_ENABLE_ASSERTIONS OR CMAKE_CONFIGURATION_TYPES)))
  set(LLVM_USE_HOST_TOOLS ON)
endif()
# when crosscompiling import the executable targets from a file
if(LLVM_USE_HOST_TOOLS)
  include(CrossCompile)
  llvm_create_cross_target(LLVM NATIVE "" Release)
endif(LLVM_USE_HOST_TOOLS)

set(LLVM_TARGET_ARCH "host"
  CACHE STRING "Set target to use for LLVM JIT or use \"host\" for automatic detection.")

  if (CMAKE_SYSTEM_NAME MATCHES "OS390")
  option(LLVM_ENABLE_THREADS "Use threads if available." OFF)
else()
  option(LLVM_ENABLE_THREADS "Use threads if available." ON)
endif()







# Add target for generating source rpm package.
# set(LLVM_SRPM_USER_BINARY_SPECFILE ${CMAKE_CURRENT_SOURCE_DIR}/llvm.spec.in
#     CACHE FILEPATH ".spec file to use for srpm generation")
# set(LLVM_SRPM_BINARY_SPECFILE ${CMAKE_CURRENT_BINARY_DIR}/llvm.spec)
# set(LLVM_SRPM_DIR "${CMAKE_CURRENT_BINARY_DIR}/srpm")




include(config-ix)

# Configure the three LLVM configuration header files.
configure_file(
  ${LLVM_MAIN_INCLUDE_DIR}/llvm/Config/config.h.cmake
  ${LLVM_INCLUDE_DIR}/llvm/Config/config.h)
configure_file(
  ${LLVM_MAIN_INCLUDE_DIR}/llvm/Config/llvm-config.h.cmake
  ${LLVM_INCLUDE_DIR}/llvm/Config/llvm-config.h)
configure_file(
  ${LLVM_MAIN_INCLUDE_DIR}/llvm/Config/abi-breaking.h.cmake
  ${LLVM_INCLUDE_DIR}/llvm/Config/abi-breaking.h)

include(HandleLLVMOptions)

include(AddLLVM)

include_directories(${LLVM_INCLUDE_DIR} ${LLVM_MAIN_INCLUDE_DIR})

add_subdirectory(external)
# Put this before tblgen. Else we have a circular dependence.
add_subdirectory(lib/Demangle)
add_subdirectory(lib/Support)
add_subdirectory(unittests)

add_subdirectory(example)