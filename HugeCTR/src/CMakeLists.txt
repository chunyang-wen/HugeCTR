# 
# Copyright (c) 2020, NVIDIA CORPORATION.
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
# 
#      http://www.apache.org/licenses/LICENSE-2.0
# 
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#

cmake_minimum_required(VERSION 3.8)
include_directories(/opt/conda/include)

include(FetchContent)

FetchContent_Declare(
  pybind11_sources
  GIT_REPOSITORY https://github.com/pybind/pybind11.git
  GIT_TAG        v2.2
)
FetchContent_GetProperties(pybind11_sources)

if(NOT pybind11_sources_POPULATED)
  FetchContent_Populate(pybind11_sources)
  add_subdirectory(
    ${pybind11_sources_SOURCE_DIR}
    ${pybind11_sources_BINARY_DIR}
    )
endif()

file(GLOB huge_ctr_src
  cpu_resource.cpp
  gpu_resource.cpp
  resource_manager.cpp
  data_simulator.cu
  data_reader.cpp
  layer.cpp
  layers/batch_norm_layer.cu
  layers/cast_layer.cu
  layers/concat_layer.cu
  layers/dropout_layer.cu
  layers/elu_layer.cu
  layers/fully_connected_layer.cu
  layers/fully_connected_layer_half.cu
  layers/fused_fully_connected_layer.cu
  layers/interaction_layer.cu
  layers/relu_layer.cu
  layers/reshape_layer.cu
  layers/sigmoid_layer.cu
  layers/slice_layer.cu
  layers/fm_order2_layer.cu
  layers/weight_multiply_layer.cu
  layers/multi_cross_layer.cu
  layers/add_layer.cu
  layers/reduce_sum_layer.cu
  layers/dot_product_layer.cu
  loss.cu
  network.cu
  network.cpp
  inference/hugectrmodel.cpp
  inference/session_inference.cpp
  inference/embedding_interface.cpp
  inference/embedding_cache.cpp
  inference/inference_utilis.cpp
  inference/parameter_server.cpp
  inference/gpu_cache/nv_gpu_cache.cu
  inference/gpu_cache/unique_op.cu
  inference/embedding_feature_combiner.cu
  inference/embedding_cache.cu
  data_readers/metadata.cpp
  metrics.cu
  optimizers/adam_optimizer.cu
  optimizers/momentum_sgd_optimizer.cu
  optimizers/nesterov_optimizer.cu
  optimizers/sgd_optimizer.cu
  optimizer.cpp
  regularizer.cu
  regularizers/l1_regularizer.cu
  regularizers/l2_regularizer.cu
  regularizers/no_regularizer.cu
  parser.cpp
  parsers/solver_parser.cpp
  parsers/learning_rate_scheduler_parser.cpp
  parsers/create_optimizer.cpp
  parsers/create_network.cpp
  parsers/create_embedding.cpp
  parsers/create_datareader.cpp
  parsers/inference_parser.cpp
  session.cpp
  plan_parser.cpp 
  data_readers/data_collector.cu
  data_readers/parquet_data_converter.cu
  hashtable/nv_hashtable.cu
  embeddings/sync_all_gpus_functor.cu
  embeddings/init_embedding_functor.cu
  embeddings/forward_per_gpu_functor.cu
  embeddings/forward_scale_functor.cu
  embeddings/forward_reorder_functor.cu
  embeddings/forward_mapping_per_gpu_functor.cu
  embeddings/forward_fuse_per_gpu_functor.cu
  embeddings/store_slot_id_functor.cu
  embeddings/backward_functor.cu
  embeddings/backward_reorder_functor.cu
  embeddings/backward_fuse_per_gpu_functor.cu
  embeddings/update_params_functor.cu
  embeddings/get_update_params_results_functor.cu
  embeddings/reduce_scatter_functor.cu
  embeddings/all_reduce_functor.cu
  embeddings/all_gather_functor.cu
  embeddings/all2all_init_forward_functor.cu
  embeddings/all2all_init_backward_functor.cu
  embeddings/all2all_forward_functor.cu
  embeddings/all2all_backward_functor.cu
  embeddings/all2all_exec_functor.cu
  embeddings/get_forward_results_functor.cu
  embeddings/get_backward_results_functor.cu
  embeddings/utils_functor.cu
  embeddings/distributed_slot_sparse_embedding_hash.cu
  embeddings/localized_slot_sparse_embedding_hash.cu
  embeddings/localized_slot_sparse_embedding_one_hot.cu
  embeddings/opt_states_functor.cu
  model_oversubscriber/localized_parameter_server_delegate.cpp
  model_oversubscriber/distributed_parameter_server_delegate.cpp
  model_oversubscriber/model_oversubscriber_impl.cpp
  model_oversubscriber/parameter_server.cpp
  model_oversubscriber/parameter_server_manager.cpp
  diagnose.cu
  ../pybind/model.cpp
  ../pybind/optimizer.cpp
  ../pybind/add_input.cpp
  ../pybind/add_sparse_embedding.cpp
  ../pybind/add_dense_layer.cpp
)

set(CMAKE_CXX_STANDARD 14)
add_library(huge_ctr_static STATIC ${huge_ctr_src})
add_library(huge_ctr_shared SHARED ${huge_ctr_src})

if(MPI_FOUND)
  target_link_libraries(huge_ctr_static PUBLIC cublas curand cudnn nccl nvToolsExt ${CMAKE_THREAD_LIBS_INIT} ${MPI_CXX_LIBRARIES} hwloc ucp ucs ucm uct cudf cudf_io cudf_base)
  target_link_libraries(huge_ctr_shared PUBLIC cublas curand cudnn nccl nvToolsExt ${CMAKE_THREAD_LIBS_INIT} ${MPI_CXX_LIBRARIES} hwloc ucp ucs ucm uct cudf cudf_io cudf_base)
  message(STATUS "${MPI_CXX_LIBRARIES}")
else()
  target_link_libraries(huge_ctr_static PUBLIC cublas curand cudnn nccl nvToolsExt ${CMAKE_THREAD_LIBS_INIT} cudf cudf_io cudf_base)
  target_link_libraries(huge_ctr_shared PUBLIC cublas curand cudnn nccl nvToolsExt ${CMAKE_THREAD_LIBS_INIT} cudf cudf_io cudf_base)
endif()

target_link_libraries(huge_ctr_static PRIVATE nlohmann_json::nlohmann_json)
target_compile_features(huge_ctr_static PUBLIC cxx_std_14)
set_target_properties(huge_ctr_static PROPERTIES CUDA_RESOLVE_DEVICE_SYMBOLS ON)
set_target_properties(huge_ctr_static PROPERTIES CUDA_ARCHITECTURES OFF)

target_link_libraries(huge_ctr_shared PRIVATE nlohmann_json::nlohmann_json)
target_compile_features(huge_ctr_shared PUBLIC cxx_std_14)
set_target_properties(huge_ctr_shared PROPERTIES CUDA_RESOLVE_DEVICE_SYMBOLS ON)
set_target_properties(huge_ctr_shared PROPERTIES CUDA_ARCHITECTURES OFF)

add_executable(huge_ctr main.cpp)
target_link_libraries(huge_ctr PUBLIC huge_ctr_static)
target_compile_features(huge_ctr PUBLIC cxx_std_14)

add_library(hugectr MODULE ../pybind/module_main.cpp)
target_link_libraries(hugectr PUBLIC pybind11::module ${PYTHON_LIBRARIES} ${CUDA_LIBRARIES} huge_ctr_shared)
set_target_properties(hugectr PROPERTIES PREFIX "")
