cmake_minimum_required(VERSION 3.13.4)

if(NOT LLVM_MAIN_SRC_DIR)
  project(spirv-to-ir-wrapper C CXX)
  find_package(LLVM REQUIRED CONFIG)
  include(AddLLVM)
  if(NOT DEFINED LLVM_SPIRV_INCLUDE_DIRS OR NOT DEFINED LLVM_SPIRV_LIB)
    message(FATAL_ERROR
      "LLVM_SPIRV_INCLUDE_DIRS and LLVM_SPIRV_LIB must be set for standalone builds")
  endif()
endif()

set(LLVM_LINK_COMPONENTS
  BinaryFormat
  Core
  Support
  )

if(LLVM_MAIN_SRC_DIR)
  # In monorepo builds SPIRVLib is an in-tree LLVM component.
  list(APPEND LLVM_LINK_COMPONENTS SPIRVLib)
endif()

if(NOT LLVM_SPIRV_INCLUDE_DIRS)
  set(LLVM_SPIRV_INCLUDE_DIRS ${LLVM_EXTERNAL_LLVM_SPIRV_SOURCE_DIR}/include)
endif()
include_directories(${LLVM_SPIRV_INCLUDE_DIRS})

add_llvm_tool(spirv-to-ir-wrapper
  spirv-to-ir-wrapper.cpp
  )

if(NOT LLVM_MAIN_SRC_DIR)
  # SPIRVLib is not an LLVM cmake component outside the monorepo; link explicitly.
  target_link_libraries(spirv-to-ir-wrapper PRIVATE ${LLVM_SPIRV_LIB})
endif()
