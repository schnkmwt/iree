# Copyright 2024 The IREE Authors
#
# Licensed under the Apache License v2.0 with LLVM Exceptions.
# See https://llvm.org/LICENSE.txt for license information.
# SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception

if(NOT IREE_TARGET_BACKEND_LLVM_CPU OR
   NOT IREE_HAL_DRIVER_LOCAL_SYNC OR
   NOT IREE_HAL_EXECUTABLE_LOADER_EMBEDDED_ELF)
  return()
endif()

## Current support is only for x86.
if(NOT IREE_ARCH STREQUAL "x86_64")
  message(STATUS "IREE mlp_pluging sample ignored -- only builds for x86_64 (today)")
  return()
endif()

# system-library plugin mechanism using the system dynamic library loader.
if(IREE_HAL_EXECUTABLE_PLUGIN_SYSTEM_LIBRARY)
  
add_library(iree_samples_custom_dispatch_cpu_mlp_plugin SHARED
  mlp_plugin.c
)
target_include_directories(iree_samples_custom_dispatch_cpu_mlp_plugin
  PRIVATE
    ${IREE_SOURCE_DIR}/runtime/src/
)

# NOTE: this is only required because we want this sample to run on all
# platforms without needing to change the library name (libfoo.so/foo.dll).
set_target_properties(iree_samples_custom_dispatch_cpu_mlp_plugin
  PROPERTIES
    WINDOWS_EXPORT_ALL_SYMBOLS ON
    PREFIX ""
    OUTPUT_NAME "mlp_plugin"
)

add_dependencies(iree-sample-deps
  iree_samples_custom_dispatch_cpu_mlp_plugin)

iree_lit_test_suite(
  NAME
    mlp_example
  SRCS
    "mlp.mlir"
  TOOLS
    FileCheck
    iree-compile
    iree-run-module
    iree_samples_custom_dispatch_cpu_mlp_plugin
  DATA
    "mlp_spec.mlir"
  LABELS
    "driver=local-sync"
    "hostonly"
)

iree_lit_test_suite(
  NAME
    mlp_tosa_example
  SRCS
    "mlp_tosa.mlir"
  TOOLS
    FileCheck
    iree-compile
    iree-run-module
    iree_samples_custom_dispatch_cpu_mlp_plugin
  DATA
    "mlp_tosa_spec.pdl.mlir"
  LABELS
    "driver=local-sync"
    "hostonly"
)

iree_lit_test_suite(
  NAME
    mlp_torch_example
  SRCS
    "mlp_torch.mlir"
  TOOLS
    FileCheck
    iree-compile
    iree-run-module
    iree_samples_custom_dispatch_cpu_mlp_plugin
  DATA
    "mlp_torch_spec.pdl.mlir"
  LABELS
    "driver=local-sync"
    "hostonly"
)

iree_lit_test_suite(
  NAME
    mlp_linalg_example
  SRCS
    "mlp_linalg.mlir"
    "mlp_linalg_two_matmul.mlir"
  TOOLS
    FileCheck
    iree-compile
    iree-run-module
    iree_samples_custom_dispatch_cpu_mlp_plugin
  DATA
    "mlp_linalg_spec.pdl.mlir"
    "mlp_spec_matmul.mlir"
  LABELS
    "driver=local-sync"
    "hostonly"
)

endif(IREE_HAL_EXECUTABLE_PLUGIN_SYSTEM_LIBRARY)
