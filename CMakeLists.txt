cmake_minimum_required(VERSION 3.10)

project(ips2ra)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake/)
include(default_build_type)

SET(CMAKE_CXX_STANDARD 17)

add_library(ips2ra INTERFACE)
target_include_directories(ips2ra INTERFACE include/)
add_subdirectory(extern/tlx)
link_libraries(ips2ra tlx)

option(IPS2RA_OPTIMIZE_FOR_NATIVE "Build with -march=native" ON)
if (IPS2RA_OPTIMIZE_FOR_NATIVE)
  include(CheckCXXCompilerFlag)
  CHECK_CXX_COMPILER_FLAG("-march=native" COMPILER_SUPPORTS_MARCH_NATIVE)
  if(COMPILER_SUPPORTS_MARCH_NATIVE)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -march=native")
    message(STATUS "IPS2RA_OPTIMIZE_FOR_NATIVE: ${IPS2RA_OPTIMIZE_FOR_NATIVE}")
  else()
    message(SEND_ERROR "IPS2RA_OPTIMIZE_FOR_NATIVE: ${IPS2RA_OPTIMIZE_FOR_NATIVE}")
  endif()
endif()

if (CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
  option(IPS2RA_USE_MCX16 "Enable 128-bit atomic instructions explicitly with the compile option -mcx16 (only required if not optimized for native)" OFF)
  if (IPS2RA_USE_MCX16)
    target_compile_options(ips2ra INTERFACE "-mcx16")
  endif()
elseif (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
  option(IPS2RA_USE_MCX16 "Enable 128-bit atomic instructions explicitly with the compile option -mcx16 (only required if not optimized for native)" OFF)
  if (IPS2RA_USE_MCX16)
    target_compile_options(ips2ra INTERFACE "-mcx16")
  endif()
endif()

option(IPS2RA_DISABLE_PARALLEL "Enable parallel code of Ips2ra" OFF)
if (NOT IPS2RA_DISABLE_PARALLEL)

  set(CMAKE_THREAD_PREFER_PTHREAD TRUE)
  set(THREADS_PREFER_PTHREAD_FLAG TRUE)
  find_package (Threads REQUIRED)
  target_link_libraries(ips2ra INTERFACE Threads::Threads)

  list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_LIST_DIR}/extern/cmake-modules)
  find_package(TBB REQUIRED)
  target_link_libraries(ips2ra INTERFACE TBB::tbb)

  target_link_libraries(ips2ra INTERFACE atomic)
  
endif()
message(STATUS "Parallel support of Ips2ra disabled: ${IPS2RA_DISABLE_PARALLEL}")

option(IPS2RA_USE_OPENMP "Enable OpenMP threads" OFF)
if (IPS2RA_USE_OPENMP)
  if (NOT IPS2RA_DISABLE_PARALLEL)
    find_package(OpenMP)
    if(OpenMP_CXX_FOUND)
      target_link_libraries(ips2ra INTERFACE OpenMP::OpenMP_CXX)
    endif()
  endif()
endif()

add_executable(ips2ra_example src/example.cpp)
target_link_libraries(ips2ra_example PRIVATE ips2ra)
