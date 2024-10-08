# Copyright (C) 2019 Intel Corporation.  All rights reserved.
# SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception

cmake_minimum_required (VERSION 3.5)

# set (CMAKE_VERBOSE_MAKEFILE 1)

if (NOT DEFINED WAMR_BUILD_PLATFORM)
  string (TOLOWER ${CMAKE_HOST_SYSTEM_NAME} WAMR_BUILD_PLATFORM)
endif ()

if (NOT WAMR_BUILD_PLATFORM STREQUAL "windows")
  project (iwasm)
else()
  project (iwasm C ASM CXX)
  enable_language (ASM_MASM)
endif()

# Reset default linker flags
set (CMAKE_SHARED_LIBRARY_LINK_C_FLAGS "")
set (CMAKE_SHARED_LIBRARY_LINK_CXX_FLAGS "")

set (CMAKE_C_STANDARD 99)

# Set WAMR_BUILD_TARGET, currently values supported:
# "X86_64", "AMD_64", "X86_32", "AARCH64[sub]", "ARM[sub]", "THUMB[sub]", "MIPS", "XTENSA"
if (NOT DEFINED WAMR_BUILD_TARGET)
  if (CMAKE_SIZEOF_VOID_P EQUAL 8)
    # Build as X86_64 by default in 64-bit platform
    set (WAMR_BUILD_TARGET "X86_64")
  else ()
    # Build as X86_32 by default in 32-bit platform
    set (WAMR_BUILD_TARGET "X86_32")
  endif ()
endif ()

if (NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif ()

if (NOT DEFINED WAMR_BUILD_INTERP)
  # Enable Interpreter by default
  set (WAMR_BUILD_INTERP 1)
endif ()

# Those three together allow exception call stacks to be dumped
if (NOT DEFINED WAMR_BUILD_THREAD_MGR)
  # Enable Interpreter by default
  set (WAMR_BUILD_THREAD_MGR 1)
endif ()
if (NOT DEFINED WAMR_BUILD_DUMP_CALL_STACK)
  # Enable Interpreter by default
  set (WAMR_BUILD_DUMP_CALL_STACK 1)
endif ()
if (NOT DEFINED WAMR_BUILD_CUSTOM_NAME_SECTION)
  # Enable Interpreter by default
  set (WAMR_BUILD_CUSTOM_NAME_SECTION 1)
endif ()

if (NOT DEFINED WAMR_BUILD_AOT)
  # Enable AOT by default.
  # TODO: Fix the build, this could be useful
  set (WAMR_BUILD_AOT 0)
endif ()

if (NOT DEFINED WAMR_BUILD_JIT)
  # Disable JIT by default.
  set (WAMR_BUILD_JIT 0)
endif ()

if (NOT DEFINED WAMR_BUILD_LIBC_BUILTIN)
  # Disable libc builtin support by default
  set (WAMR_BUILD_LIBC_BUILTIN 0)
endif ()

if (NOT DEFINED WAMR_BUILD_LIBC_WASI)
  # Disable libc wasi support by default
  set (WAMR_BUILD_LIBC_WASI 0)
endif ()

if (NOT DEFINED WAMR_BUILD_FAST_INTERP)
  # Enable fast interpreter
  set (WAMR_BUILD_FAST_INTERP 1)
endif ()

if (NOT DEFINED WAMR_BUILD_MULTI_MODULE)
  # Enable multiple modules
  set (WAMR_BUILD_MULTI_MODULE 0)
endif ()

if (NOT DEFINED WAMR_BUILD_LIB_PTHREAD)
  # Disable pthread library by default
  set (WAMR_BUILD_LIB_PTHREAD 0)
endif ()

if (NOT DEFINED WAMR_BUILD_MINI_LOADER)
  # Disable wasm mini loader by default
  set (WAMR_BUILD_MINI_LOADER 0)
endif ()

if (NOT DEFINED WAMR_BUILD_SIMD)
  # Disable SIMD by default
  set (WAMR_BUILD_SIMD 0)
endif ()

if (COLLECT_CODE_COVERAGE EQUAL 1)
  set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fprofile-arcs -ftest-coverage")
endif ()

set (WAMR_ROOT_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../wasm-micro-runtime)

include (${WAMR_ROOT_DIR}/build-scripts/runtime_lib.cmake)
# vmlib/VMcore seems to be the static linkage version of iwasm, this is easier to embed.
add_library(vmlib ${WAMR_RUNTIME_LIB_SOURCE})

set (CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -Wl,--gc-sections -pie -fPIE")

set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -Wextra -Wformat -Wformat-security")

if (WAMR_BUILD_TARGET MATCHES "X86_.*" OR WAMR_BUILD_TARGET STREQUAL "AMD_64")
  if (NOT (CMAKE_C_COMPILER MATCHES ".*clang.*" OR CMAKE_C_COMPILER_ID MATCHES ".*Clang"))
    set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -mindirect-branch-register")
  endif ()
endif ()

include (${SHARED_DIR}/utils/uncommon/shared_uncommon.cmake)

if (WAMR_BUILD_PLATFORM STREQUAL "windows")
  # Disable export macros for the static lib.
  target_compile_definitions(vmlib PRIVATE WASM_API_EXTERN=)
  # Append an empty generator expression to prevent config sub-directories to be used.
  # https://stackoverflow.com/a/56201564/56315
  set_target_properties(vmlib PROPERTIES ARCHIVE_OUTPUT_DIRECTORY $<0:>)
endif()
