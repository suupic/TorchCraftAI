# Copyright (c) 2017-present, Facebook, Inc.
#
# This source code is licensed under the MIT license found in the
# LICENSE file in the root directory of this source tree.

FIND_PACKAGE(ZMQ REQUIRED)

FIND_PACKAGE(CUDA)
IF(CUDA_FOUND)
  ADD_DEFINITIONS(-DHAVE_CUDA)
ENDIF(CUDA_FOUND)

FILE(GLOB TRSRCS *.cpp)
LIST(APPEND SRCS ${TRSRCS})

ADD_LIBRARY(cpid "${SRCS}")
TARGET_COMPILE_OPTIONS(cpid PRIVATE "${CHERRYPI_WARNINGS}")
SET_PROPERTY(TARGET cpid PROPERTY POSITION_INDEPENDENT_CODE ON)
FILE(GLOB TRSRCS *.cpp)

SET(C10D_INCLUDES
  "${PROJECT_SOURCE_DIR}/3rdparty/pytorch/lib/tmp_install/include"
  )

IF(CUDA_FOUND)
  FIND_LIBRARY(GLOO_CUDA gloo_cuda
    PATHS "${PROJECT_SOURCE_DIR}/3rdparty/pytorch/torch/lib/tmp_install/lib"
    NO_DEFAULT_PATH
  )
  TARGET_LINK_LIBRARIES(cpid cudart "${GLOO_CUDA}")
  TARGET_INCLUDE_DIRECTORIES(cpid SYSTEM PUBLIC ${CUDA_TOOLKIT_INCLUDE})
ENDIF(CUDA_FOUND)

IF(NOT TORCH_FOUND)
  FIND_PACKAGE(Torch REQUIRED)
  ADD_DEFINITIONS(-DHAVE_ATEN)
ENDIF(NOT TORCH_FOUND)

FIND_LIBRARY(C10D c10d
  PATHS "${PROJECT_SOURCE_DIR}/3rdparty/pytorch/torch/lib/tmp_install/lib"
  NO_DEFAULT_PATH
)
FIND_LIBRARY(GLOO gloo
  PATHS "${PROJECT_SOURCE_DIR}/3rdparty/pytorch/torch/lib/tmp_install/lib"
  NO_DEFAULT_PATH
)
FIND_LIBRARY(GLOO_BUILDER gloo_builder
  PATHS "${PROJECT_SOURCE_DIR}/3rdparty/pytorch/torch/lib/tmp_install/lib"
  NO_DEFAULT_PATH
)
TARGET_LINK_LIBRARIES(cpid 
  "${C10D}"
  "${GLOO}"
  "${GLOO_BUILDER}"
  glog Torch fmt ${ZMQ_LIBRARIES} visdom pthread nccl
  )
TARGET_INCLUDE_DIRECTORIES(cpid PUBLIC "${PROJECT_SOURCE_DIR}/3rdparty/include")
TARGET_INCLUDE_DIRECTORIES(cpid PRIVATE "${ZMQ_INCLUDE_DIR}")
TARGET_INCLUDE_DIRECTORIES(cpid PUBLIC "${PROJECT_SOURCE_DIR}")
