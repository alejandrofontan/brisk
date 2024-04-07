cmake_minimum_required(VERSION 3.16)
project(brisk)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -O3 -march=native")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -O3 -march=native")
set (CMAKE_CXX_STANDARD 17)

add_compile_options(-Wextra -Wpedantic)
set(SRCS_BRISK
  				brisk/src/brisk-descriptor-extractor.cc
                               brisk/src/brisk-feature-detector.cc
                               brisk/src/brisk-layer.cc
                               brisk/src/brisk-opencv.cc
                               brisk/src/brisk-scale-space.cc
                               brisk/src/brute-force-matcher.cc
                               brisk/src/harris-feature-detector.cc
                               brisk/src/harris-score-calculator.cc
                               brisk/src/harris-score-calculator-float.cc
                               brisk/src/harris-scores.cc
                               brisk/src/image-down-sampling.cc
                               brisk/src/pattern-provider.cc
                               brisk/src/vectorized-filters.cc
                               brisk/src/test/image-io.cc
                               brisk/src/timer.cc
  )
file(GLOB SRCS_AGAST  "${PROJECT_SOURCE_DIR}/agast/src/*.cc")

file(GLOB HDRS_BRISK   "${PROJECT_SOURCE_DIR}/brisk/include/brisk/*.h"
			"${PROJECT_SOURCE_DIR}/brisk/include/brisk/cameras/*.h"
			"${PROJECT_SOURCE_DIR}/brisk/include/brisk/cameras/implementation/*.h"
			"${PROJECT_SOURCE_DIR}/brisk/include/brisk/internal/*.h "
			 )
file(GLOB HDRS_AGAST  "${PROJECT_SOURCE_DIR}/agast/include/*.h" 
		       "${PROJECT_SOURCE_DIR}/agast/include/agast/*.h")
	
find_package(OpenCV REQUIRED)
find_package(glog REQUIRED)

add_library(${PROJECT_NAME} SHARED ${SRCS_BRISK} ${SRCS_AGAST})
target_include_directories(${PROJECT_NAME} PUBLIC 
	"${PROJECT_SOURCE_DIR}/brisk/include/brisk"
	"${PROJECT_SOURCE_DIR}/agast/include"
	"${PROJECT_SOURCE_DIR}/agast/include/agast"
)

target_link_libraries(${PROJECT_NAME} ${OpenCV_LIBRARIES} glog::glog)

configure_file(brisk/src/brisk.cmake.in
  "${PROJECT_BINARY_DIR}/briskConfig.cmake" @ONLY)

install(TARGETS ${PROJECT_NAME}
        RUNTIME DESTINATION  ${CMAKE_INSTALL_PREFIX}/bin
        LIBRARY DESTINATION  ${CMAKE_INSTALL_PREFIX}/lib
        ARCHIVE DESTINATION  ${CMAKE_INSTALL_PREFIX}/lib)
        
install(DIRECTORY
	${HDRS_BRISK}
	${HDRS_AGAST}	
        DESTINATION  ${CMAKE_INSTALL_PREFIX}/include/brisk
        FILES_MATCHING PATTERN "*.h" PATTERN "*.hpp")
        
install(FILES "${PROJECT_BINARY_DIR}/briskConfig.cmake" 
  DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/cmake/brisk/)  
  

