cmake_minimum_required(VERSION 3.16)
project(brisk)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -O3 -march=native")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -O3 -march=native")
set (CMAKE_CXX_STANDARD 17)

add_compile_options(-Wextra -Wpedantic)

if(${CMAKE_HOST_SYSTEM_PROCESSOR} MATCHES "arm*")
  message(STATUS "ARM processor detected, will attempt to use NEON.")
  add_definitions(-mfpu=neon -DHAVE_OPENCV -Wall -pedantic -flax-vector-conversions -fPIC)
else()
  message(STATUS "Assuming SSE instructions available.")
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

configure_file(scr/brisk.cmake.in
  "${PROJECT_BINARY_DIR}/briskConfig.cmake" @ONLY)

install(TARGETS ${PROJECT_NAME}
        RUNTIME DESTINATION  ${CMAKE_INSTALL_PREFIX}/bin
        LIBRARY DESTINATION  ${CMAKE_INSTALL_PREFIX}/lib
        ARCHIVE DESTINATION  ${CMAKE_INSTALL_PREFIX}/lib)
        
install(DIRECTORY
	"${PROJECT_SOURCE_DIR}/brisk/include/brisk"
	"${PROJECT_SOURCE_DIR}/brisk/include/sse2neon"
	"${PROJECT_SOURCE_DIR}/agast/include/agast"
        DESTINATION  ${CMAKE_INSTALL_PREFIX}/include
        FILES_MATCHING PATTERN "*.h" PATTERN "*.hpp")

install(DIRECTORY
	"${PROJECT_SOURCE_DIR}/brisk/include/brisk/internal"
        DESTINATION  ${CMAKE_INSTALL_PREFIX}/include/brisk
        FILES_MATCHING PATTERN "*.h" PATTERN "*.hpp")
                                            
install(FILES "${PROJECT_BINARY_DIR}/briskConfig.cmake" 
  DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/cmake/brisk/)  
  

