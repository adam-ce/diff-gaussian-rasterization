#
# Copyright (C) 2023, Inria
# GRAPHDECO research group, https://team.inria.fr/graphdeco
# All rights reserved.
#
# This software is free for non-commercial, research and evaluation use 
# under the terms of the LICENSE.md file.
#
# For inquiries contact  george.drettakis@inria.fr
#

cmake_minimum_required(VERSION 3.16)

project(DiffRast LANGUAGES CUDA CXX)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
if(NOT DEFINED CMAKE_CUDA_ARCHITECTURES)
  set(CMAKE_CUDA_ARCHITECTURES "75;86;90")
endif()

include(FetchContent)
if(AUTOUPDATE_STROKE_LIBRARY)
    FetchContent_Declare(stroke
        GIT_REPOSITORY  git@github.com:cg-tuwien/cuda_stroke.git
        GIT_TAG         origin/main
        SOURCE_DIR      ${CMAKE_SOURCE_DIR}/3rdparty/stroke
        SOURCE_SUBDIR   src
    )
    FetchContent_MakeAvailable(stroke)
else()
    if (NOT EXISTS ${CMAKE_SOURCE_DIR}/3rdparty/stroke)
        FetchContent_Declare(stroke
            GIT_REPOSITORY  git@github.com:cg-tuwien/cuda_stroke.git
            GIT_TAG         main
            SOURCE_DIR      ${CMAKE_SOURCE_DIR}/3rdparty/stroke
        )
        FetchContent_MakeAvailable(stroke)
    elseif(NOT TARGET stroke)
        add_subdirectory(${CMAKE_SOURCE_DIR}/3rdparty/stroke ${CMAKE_BINARY_DIR}/_deps/stroke-build)
    endif()
endif()

add_library(CudaRasterizer
	cuda_rasterizer/config.h
	cuda_rasterizer/backward.h
	cuda_rasterizer/backward.cu
	cuda_rasterizer/forward.h
	cuda_rasterizer/forward.cu
	cuda_rasterizer/auxiliary.h
	cuda_rasterizer/rasterizer_impl.cu
	cuda_rasterizer/rasterizer_impl.h
	cuda_rasterizer/rasterizer.h
    cuda_rasterizer/utils.h
    cuda_rasterizer/grad/utils.h
)

target_compile_definitions(CudaRasterizer INTERFACE GLM_FORCE_LEFT_HANDED)
target_link_libraries(CudaRasterizer PUBLIC stroke)
set_target_properties(CudaRasterizer PROPERTIES CUDA_ARCHITECTURES "70;75;86")
target_compile_features(CudaRasterizer INTERFACE cuda_std_14 cxx_std_17)
target_include_directories(CudaRasterizer PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/cuda_rasterizer)
target_include_directories(CudaRasterizer PRIVATE ${CMAKE_CUDA_TOOLKIT_INCLUDE_DIRECTORIES})
