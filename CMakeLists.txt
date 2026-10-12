cmake_minimum_required(VERSION 3.16)
project(brisk)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -O3")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -O3")
# Off by default so the library runs on any CPU of the target architecture (e.g. the conda package);
# turn it on for a local build tuned to this machine.
option(BRISK_NATIVE "Optimise for the build machine's CPU (-march=native)" OFF)
if(BRISK_NATIVE)
  add_compile_options(-march=native)
endif()
set (CMAKE_CXX_STANDARD 17)

add_compile_options(-Wextra -Wpedantic)

# SIMD: the SSE code paths are the reference; on ARM they run through include/sse2neon (sse2neon v1.8.0, MIT) plus
# the native NEON paths behind `defined(__ARM_NEON) || defined(__ARM_NEON__)` (AArch64 compilers define __ARM_NEON,
# 32-bit ARM ones __ARM_NEON__; Apple's arm64 clang defines both). CMAKE_SYSTEM_PROCESSOR is the target, which is
# what matters when cross-compiling (conda packages).
# -flax-vector-conversions: the NEON paths pass int16x8_t / uint8x16_t values to sse2neon's __m128i (an int64x2_t);
# clang allows such same-size integer vector conversions by default, GCC only with this flag.
if(CMAKE_SYSTEM_PROCESSOR MATCHES "^(aarch64|arm64|ARM64)$")
  message(STATUS "AArch64 target: NEON is part of the base ISA, SSE intrinsics through sse2neon.")
  add_definitions(-DHAVE_OPENCV -Wall -pedantic -flax-vector-conversions -fPIC)
elseif(CMAKE_SYSTEM_PROCESSOR MATCHES "^arm")
  message(STATUS "32-bit ARM target: NEON through -mfpu=neon, SSE intrinsics through sse2neon.")
  add_definitions(-mfpu=neon -DHAVE_OPENCV -Wall -pedantic -flax-vector-conversions -fPIC)
else()
  message(STATUS "x86-64 target: SSSE3 instructions assumed.")
  add_definitions(-mssse3 -DHAVE_OPENCV -Wall -pedantic -fPIC)
endif()

foreach(dir ${dirs})
    set(HAVE_OPENCV_NONFREE_H TRUE)
endforeach()

set(SRCS_BRISK
	src/brisk-descriptor-extractor.cc
        src/brisk-feature-detector.cc   
        src/brisk-layer.cc
        src/brisk-scale-space.cc
        src/brute-force-matcher.cc   
        src/harris-feature-detector.cc
        src/harris-score-calculator.cc
        src/harris-score-calculator-float.cc
        src/harris-scores.cc
        src/image-down-sampling.cc
        src/pattern-provider.cc
        src/vectorized-filters.cc
  )
file(GLOB SRCS_AGAST  "${PROJECT_SOURCE_DIR}/agast/src/*.cc")

find_package(OpenCV REQUIRED)

add_library(${PROJECT_NAME} SHARED ${SRCS_BRISK} ${SRCS_AGAST})
target_include_directories(${PROJECT_NAME} PUBLIC 
	"${PROJECT_SOURCE_DIR}/include"
	"${PROJECT_SOURCE_DIR}/agast/include"
)

target_link_libraries(${PROJECT_NAME} ${OpenCV_LIBRARIES})

configure_file(src/brisk.cmake.in
  "${PROJECT_BINARY_DIR}/briskConfig.cmake" @ONLY)

install(TARGETS ${PROJECT_NAME}
        RUNTIME DESTINATION  ${CMAKE_INSTALL_PREFIX}/bin
        LIBRARY DESTINATION  ${CMAKE_INSTALL_PREFIX}/lib
        ARCHIVE DESTINATION  ${CMAKE_INSTALL_PREFIX}/lib)
        
install(DIRECTORY
	"${PROJECT_SOURCE_DIR}/include/brisk"
	"${PROJECT_SOURCE_DIR}/include/sse2neon"
	"${PROJECT_SOURCE_DIR}/agast/include/agast"
        DESTINATION  ${CMAKE_INSTALL_PREFIX}/include
        FILES_MATCHING PATTERN "*.h" PATTERN "*.hpp")

install(DIRECTORY
	"${PROJECT_SOURCE_DIR}/include/brisk/internal"
        DESTINATION  ${CMAKE_INSTALL_PREFIX}/include/brisk
        FILES_MATCHING PATTERN "*.h" PATTERN "*.hpp")
                                            
install(FILES "${PROJECT_BINARY_DIR}/briskConfig.cmake" 
  DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/cmake/brisk/)  
  

