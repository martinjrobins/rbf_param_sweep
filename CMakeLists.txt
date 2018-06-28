cmake_minimum_required(VERSION 2.8)
project(Test)

include(CMakeToolsHelpers OPTIONAL)

set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}")

list(APPEND CMAKE_CXX_FLAGS "-Wall -Wno-deprecated -std=c++14 -ftemplate-backtrace-limit=0")

find_package(Thrust REQUIRED)
find_package(CUDA REQUIRED)

add_definitions(-DHAVE_THRUST)

option(Aboria_THRUST_USE_THRUST_TUPLE "Use thrust tuple everywhere, even for std::vector" OFF)
if (Aboria_THRUST_USE_THRUST_TUPLE)
    add_definitions(-DABORIA_THRUST_USE_THRUST_TUPLE)
endif()

option(Aboria_THRUST_TEST_NVCC "Turn on tests using nvcc" OFF)
if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
    option(Aboria_THRUST_TEST_CUDA_CLANG "Turn on tests using cuda clang" OFF)
endif()
#http://stackoverflow.com/questions/34996295/trying-to-get-cuda-7-5-to-work-with-gcc-5-x

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

set(aboria_dirs /scratch/robinsonm/git/Aboria/src /scratch/robinsonm/git/Aboria/third-party)
include_directories(SYSTEM ${H2Lib_INCLUDE_DIRS} ${aboria_dirs} ${EIGEN3_INCLUDE_DIR} ${THRUST_INCLUDE_DIR})

add_executable(gaussian gaussian.cpp)
add_executable(matern matern.cpp)
add_executable(exponential exponential.cpp)

cuda_add_executable(gaussian_nvcc gaussian.cpp)
cuda_add_executable(matern_nvcc matern.cpp)
cuda_add_executable(exponential_nvcc exponential.cpp)

target_link_libraries(gaussian ${H2Lib_LIBRARIES})
target_link_libraries(matern ${H2Lib_LIBRARIES})
target_link_libraries(exponential ${H2Lib_LIBRARIES})

target_link_libraries(gaussian_nvcc ${H2Lib_LIBRARIES})
target_link_libraries(matern_nvcc ${H2Lib_LIBRARIES})
target_link_libraries(exponential_nvcc ${H2Lib_LIBRARIES})


