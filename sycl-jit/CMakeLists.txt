cmake_minimum_required(VERSION 3.20.0 FATAL_ERROR)

# Standalone build support
if(NOT LLVM_MAIN_SRC_DIR)
  find_package(LLVM REQUIRED CONFIG)
  include(AddLLVM)
  find_package(Clang REQUIRED CONFIG)
  find_package(Python3 REQUIRED)
  set(CMAKE_INCLUDE_CURRENT_DIR ON)
endif()

# Define a variable holding the root directory of the JIT compiler project
# for use in includes etc.
set(SYCL_JIT_BASE_DIR ${CMAKE_CURRENT_SOURCE_DIR})

# For some reason, the LLVMSPIRVLib does not define any public includes.
# To link against the library, define the following link to its include
# directories, similar to how clang/CMakeLists.txt does it.
if(NOT LLVM_SPIRV_INCLUDE_DIRS)
  set(LLVM_SPIRV_INCLUDE_DIRS "${LLVM_MAIN_SRC_DIR}/../llvm-spirv/include")
endif()

# In standalone builds, sycl-headers target doesn't exist.
if(NOT TARGET sycl-headers)
  add_custom_target(sycl-headers)
endif()

if (NOT WIN32 AND NOT CYGWIN)
  # Set library-wide warning options.
  set(SYCL_JIT_WARNING_FLAGS -Wall -Wextra -Wconversion -Wimplicit-fallthrough)

  option(SYCL_JIT_ENABLE_WERROR "Treat all warnings as errors in SYCL JIT library" ON)
  if(SYCL_JIT_ENABLE_WERROR)
    list(APPEND SYCL_JIT_WARNING_FLAGS -Werror)
  endif(SYCL_JIT_ENABLE_WERROR)
endif()


add_subdirectory(jit-compiler)
add_subdirectory(passes)

# Loadable plugins for opt aren't supported on Windows,
# so we can't execute the tests.
if (NOT WIN32 AND NOT CYGWIN)
  add_subdirectory(test)
endif()
