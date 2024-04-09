# Copyright (c) Meta Platforms, Inc. and affiliates.
# All rights reserved.
#
# This source code is licensed under the MIT-style license found in the
# LICENSE file in the root directory of this source tree.

set(DIETGPU_BACKEND "CUDA" CACHE STRING "Select the desired backend: CUDA/cuda and HIP/hip.")

if(DIETGPU_BACKEND STREQUAL "CUDA" OR DIETGPU_BACKEND STREQUAL "cuda")

  cmake_minimum_required(VERSION 3.10 FATAL_ERROR)
  project(dietgpu LANGUAGES CUDA CXX VERSION 1.0)
  set(CMAKE_CXX_STANDARD 17)
  set(CMAKE_C_STANDARD 11)

  include(CheckLanguage)
  check_language(CUDA)

  if(NOT DEFINED CMAKE_CUDA_STANDARD)
    set(CMAKE_CUDA_STANDARD 17)
    set(CMAKE_CUDA_STANDARD_REQUIRED ON)
  endif()

  find_package(CUDA REQUIRED)

  if(${CMAKE_VERSION} VERSION_LESS_EQUAL "3.13.4")
    cuda_select_nvcc_arch_flags(ARCH_FLAGS "Auto")
    message("ARCH_FLAGS = ${ARCH_FLAGS}")
    string(REPLACE "-gencode;" "--generate-code=" ARCH_FLAGS "${ARCH_FLAGS}")
    string(APPEND CMAKE_CUDA_FLAGS "${ARCH_FLAGS}")
  else()
    include(FindCUDA/select_compute_arch)
    CUDA_DETECT_INSTALLED_GPUS(INSTALLED_GPU_CCS_1)
    string(STRIP "${INSTALLED_GPU_CCS_1}" INSTALLED_GPU_CCS_2)
    string(REPLACE " " ";" INSTALLED_GPU_CCS_3 "${INSTALLED_GPU_CCS_2}")
    string(REPLACE "." "" CUDA_ARCH_LIST "${INSTALLED_GPU_CCS_3}")
    set(CMAKE_CUDA_ARCHITECTURES ${CUDA_ARCH_LIST})
    set_property(GLOBAL PROPERTY CUDA_ARCHITECTURES "${CUDA_ARCH_LIST}")
  endif()

  # Set default build type.
  if(NOT CMAKE_BUILD_TYPE)
    message(STATUS "Setting build type to 'RelWithDebInfo' as none was specified.")
    set(CMAKE_BUILD_TYPE "RelWithDebInfo" CACHE STRING
      "Choose the type of build, options are: Debug Release RelWithDebInfo MinSizeRel."
      FORCE
    )
  endif()

  set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
  set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
  set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

  add_subdirectory(third_party/glog)
  add_subdirectory(third_party/googletest)

  add_subdirectory(dietgpu)
  add_subdirectory(dietgpu/utils)
  add_subdirectory(dietgpu/ans)
  add_subdirectory(dietgpu/float)

elseif(DIETGPU_BACKEND STREQUAL "HIP" OR DIETGPU_BACKEND STREQUAL "hip")

  cmake_minimum_required(VERSION 3.21 FATAL_ERROR)
  project(dietgpu_hip LANGUAGES HIP CXX VERSION 1.0)
  set(CMAKE_CXX_STANDARD 17)
  set(CMAKE_C_STANDARD 11)

  include(CheckLanguage)
  check_language(HIP)

  if(NOT DEFINED CMAKE_HIP_STANDARD)
    set(CMAKE_HIP_STANDARD 17)
    set(CMAKE_HIP_STANDARD_REQUIRED ON)
  endif()

  find_package(hip REQUIRED)
  find_package(rocthrust REQUIRED)
  if(rocthrust_FOUND)
    message("[psz::info] rocthrust FOUND")
    message("[psz::info] $\{rocthrust_INCLUDE_DIRS\}: " ${rocthrust_INCLUDE_DIRS})
    message("[psz::info] $\{rocthrust_LIBRARIES\}: " ${rocthrust_LIBRARIES})
    include_directories(${rocthrust_INCLUDE_DIRS})
  endif()
  
  find_package(rocprim REQUIRED)
  if(rocprim_FOUND)
    message("[psz::info] rocprim FOUND")
    message("[psz::info] $\{rocprim_INCLUDE_DIRS\}: " ${rocprim_INCLUDE_DIRS})
    message("[psz::info] $\{rocprim_LIBRARIES\}: " ${rocprim_LIBRARIES})
    include_directories(${rocprim_INCLUDE_DIRS})
  endif()

  #   include(FindCUDA/select_compute_arch)
  #   CUDA_DETECT_INSTALLED_GPUS(INSTALLED_GPU_CCS_1)
  #   string(STRIP "${INSTALLED_GPU_CCS_1}" INSTALLED_GPU_CCS_2)
  #   string(REPLACE " " ";" INSTALLED_GPU_CCS_3 "${INSTALLED_GPU_CCS_2}")
  #   string(REPLACE "." "" CUDA_ARCH_LIST "${INSTALLED_GPU_CCS_3}")
  #   set(CMAKE_CUDA_ARCHITECTURES ${CUDA_ARCH_LIST})
  #   set_property(GLOBAL PROPERTY CUDA_ARCHITECTURES "${CUDA_ARCH_LIST}")

  # Set default build type.
  if(NOT CMAKE_BUILD_TYPE)
    message(STATUS "Setting build type to 'RelWithDebInfo' as none was specified.")
    set(CMAKE_BUILD_TYPE "RelWithDebInfo" CACHE STRING
      "Choose the type of build, options are: Debug Release RelWithDebInfo MinSizeRel."
      FORCE
    )
  endif()

  set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
  set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
  set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

  add_subdirectory(third_party/glog)
  add_subdirectory(third_party/googletest)

  add_subdirectory(dietgpu_hip)
  add_subdirectory(dietgpu_hip/utils_hip)
  add_subdirectory(dietgpu_hip/ans_hip)
  add_subdirectory(dietgpu_hip/float_hip)

endif()