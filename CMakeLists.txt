cmake_minimum_required(VERSION 2.8)
project(Test)

include(CMakeToolsHelpers OPTIONAL)

set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}")

list(APPEND CMAKE_CXX_FLAGS "-Wall -Wno-deprecated -std=c++14 -ftemplate-backtrace-limit=0")

find_package(Boost 1.50.0 REQUIRED serialization)
find_package(Thrust REQUIRED)
find_package(CUDA REQUIRED)

add_definitions(-DHAVE_THRUST)

add_definitions(-D_MWAITXINTRIN_H_INCLUDED)
set(
    CUDA_NVCC_FLAGS
    ${CUDA_NVCC_FLAGS};
    --expt-relaxed-constexpr
    -std=c++14
    #-G
    #-g
    #--keep
    #-lineinfo
    --expt-extended-lambda
    -ftemplate-backtrace-limit=0
    #-gencode arch=compute_30,code=sm_30
    #-gencode arch=compute_35,code=sm_35
    #-gencode arch=compute_50,code=sm_50
    #-gencode arch=compute_52,code=sm_52
    -gencode arch=compute_50,code=sm_50
    )
#   if (CMAKE_BUILD_TYPE STREQUAL Debug)
#       set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS};-G)
#   endif()


find_package(OpenMP REQUIRED)
add_definitions(-DHAVE_OPENMP)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenMP_EXE_LINKER_FLAGS}")

find_package(Eigen3 REQUIRED)
add_definitions(-DHAVE_EIGEN)
set( CMAKE_EXPORT_COMPILE_COMMANDS 1 )

find_package(H2Lib REQUIRED)
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${H2Lib_LINKER_FLAGS}")
add_definitions(-DHAVE_H2LIB)

set(aboria_dirs Aboria/src Aboria/third-party)

list(APPEND Aboria_LIBRARIES "")
include_directories(SYSTEM ${Boost_INCLUDE_DIR} ${H2Lib_INCLUDE_DIRS} ${aboria_dirs} ${EIGEN3_INCLUDE_DIR} ${THRUST_INCLUDE_DIR})

if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
    add_executable(test_cuda_clang test.cpp)
    target_compile_options(test_cuda_clang PRIVATE "--cuda-gpu-arch=sm_50" "-xcuda" "-pthread")
    target_link_libraries(test_cuda_clang ${Boost_LIBRARIES} ${H2Lib_LIBRARIES} ${CUDA_LIBRARIES} cudart_static dt rt)
endif()

add_executable(gaussian gaussian.cpp)
add_executable(matern matern.cpp)
add_executable(exponential exponential.cpp)

add_executable(test test.cpp)
cuda_add_executable(test_nvcc test.cu)

cuda_add_executable(gaussian_nvcc gaussian.cu)
cuda_add_executable(matern_nvcc matern.cu)
cuda_add_executable(exponential_nvcc exponential.cu)

target_link_libraries(gaussian ${H2Lib_LIBRARIES} ${Boost_LIBRARIES})
target_link_libraries(matern ${H2Lib_LIBRARIES} ${Boost_LIBRARIES})
target_link_libraries(exponential ${H2Lib_LIBRARIES} ${Boost_LIBRARIES})

target_link_libraries(gaussian_nvcc ${H2Lib_LIBRARIES} ${Boost_LIBRARIES})
target_link_libraries(matern_nvcc ${H2Lib_LIBRARIES} ${Boost_LIBRARIES})
target_link_libraries(exponential_nvcc ${H2Lib_LIBRARIES} ${Boost_LIBRARIES})


