find_package(Torch REQUIRED)

add_library(dietgpu_hip SHARED
  DietGpu.cpp.hip
)
add_dependencies(dietgpu_hip
  gpu_float_compress
)
target_link_libraries(dietgpu_hip PRIVATE
  gpu_float_compress
  glog::glog
)
target_include_directories(dietgpu_hip PRIVATE
  $<BUILD_INTERFACE:${dietgpu_SOURCE_DIR}>
  "${TORCH_INCLUDE_DIRS}"
)
