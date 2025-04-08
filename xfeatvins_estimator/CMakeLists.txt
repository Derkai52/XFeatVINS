cmake_minimum_required(VERSION 2.8.3)
project(xfeatvins)

set(CMAKE_BUILD_TYPE "Release")
set(CMAKE_CXX_STANDARD 17)
#-DEIGEN_USE_MKL_ALL")
set(CMAKE_CXX_FLAGS_RELEASE "-O3 -Wall -g")

find_package(catkin REQUIRED COMPONENTS
    roscpp
    std_msgs
    geometry_msgs
    nav_msgs
    tf
    cv_bridge
    camera_models
    image_transport)

find_package(OpenCV REQUIRED)
find_package(Ceres REQUIRED)
find_package(CUDA REQUIRED)
find_package(Torch REQUIRED PATHS /home/emnavi/.local/lib/python3.8/site-packages/torch/share/cmake/Torch)
message(STATUS "Found Torch include is ${TORCH_INCLUDE_DIRS}")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${TORCH_CXX_FLAGS}")

# TensorRT paths 
list(APPEND PLUGINS "nvinfer")
list(APPEND PLUGINS "nvonnxparser")
list(APPEND PLUGINS "nvparsers")
list(APPEND PLUGINS "nvinfer_plugin")

foreach(libName ${PLUGINS})
    find_library(${libName}_lib NAMES ${libName} "/usr" PATH_SUFFIXES lib)
    list(APPEND PLUGIN_LIBS "${${libName}_lib}")
endforeach()

# Include directories
include_directories(include/ ${CUDA_INCLUDE_DIRS} ${TENSORRT_INCLUDE_DIR} ${OpenCV_INCLUDE_DIRS} ${TORCH_INCLUDE_DIRS})

# Create XFeat library
add_library(xfeat_lib SHARED
    src/featureTracker/xfeat.cpp
    src/featureTracker/xfeat_include/xfeat.h 
    src/featureTracker/xfeat_include/InterpolateSparse2D.h   
)

# Link directories
link_directories(${CUDA_LIBRARY_DIRS} ${TENSORRT_LIBRARY_DIR})
target_link_libraries(xfeat_lib ${CUDA_LIBRARIES} ${OpenCV_LIBS} ${TORCH_LIBRARIES} nvinfer nvonnxparser nvinfer_plugin)

# include_directories(${PROJECT_SOURCE_DIR}/featureTracker/ort_include/)
include_directories(${PROJECT_SOURCE_DIR}/featureTracker/xfeat_include/)
include_directories(${PROJECT_SOURCE_DIR}/src/estimator/)
include_directories(${catkin_INCLUDE_DIRS} ${CERES_INCLUDE_DIRS})

set(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)
find_package(Eigen3)
include_directories(
  ${catkin_INCLUDE_DIRS}
  ${EIGEN3_INCLUDE_DIR}
)

catkin_package()

add_library(xfeatvins_lib
    src/estimator/parameters.cpp
    src/estimator/estimator.cpp
    src/estimator/feature_manager.cpp
    src/factor/pose_local_parameterization.cpp
    src/factor/projectionTwoFrameOneCamFactor.cpp
    src/factor/projectionTwoFrameTwoCamFactor.cpp
    src/factor/projectionOneFrameTwoCamFactor.cpp
    src/factor/marginalization_factor.cpp
    src/utility/utility.cpp
    src/utility/visualization.cpp
    src/utility/CameraPoseVisualization.cpp
    src/initial/solve_5pts.cpp
    src/initial/initial_aligment.cpp
    src/initial/initial_sfm.cpp
    src/initial/initial_ex_rotation.cpp
    src/featureTracker/feature_tracker.cpp
    src/featureTracker/feature_tracker_dpl.cpp
    # src/featureTracker/extractor_matcher_dpl.cpp
    src/featureTracker/transform_dpl.cpp)
target_link_libraries(xfeatvins_lib ${catkin_LIBRARIES} ${OpenCV_LIBS} ${CERES_LIBRARIES} xfeat_lib ${TORCH_LIBRARIES})


get_filename_component(PROJECT_SOURCE_DIR_ABS "${CMAKE_SOURCE_DIR}" ABSOLUTE)
add_executable(xfeatvins_node src/xfeatvins_main.cpp)
target_compile_definitions(xfeatvins_node PRIVATE PROJECT_SOURCE_DIR="${PROJECT_SOURCE_DIR_ABS}")
target_link_libraries(xfeatvins_node xfeatvins_lib) 

# add_executable(kitti_odom_test src/KITTIOdomTest.cpp )
# target_link_libraries(kitti_odom_test vins_lib -pthread) 

# add_executable(kitti_gps_test src/KITTIGPSTest.cpp)
# target_link_libraries(kitti_gps_test vins_lib -pthread) 

