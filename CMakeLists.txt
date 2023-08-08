CMAKE_MINIMUM_REQUIRED (VERSION 3.4)
#######################################
# user config

# Based on the ARM_ARCHITECTURE environment variable
# Set IS_ARM to build on the Odroid board
MESSAGE("ARM_ARCHITECTURE environment variable set to " ${CMAKE_SYSTEM_PROCESSOR})

if(${CMAKE_SYSTEM_PROCESSOR} MATCHES "arm")
    SET(IS_ARM TRUE)
    # Set any ARM-specific build flags here.
else()
    message(STATUS "Building on non-ARM architecture")
    SET(IS_ARM FALSE)
endif()

IF(DEFINED ENV{ARM_ARCHITECTURE})
  MESSAGE("ARM_ARCHITECTURE environment variable set to " $ENV{ARM_ARCHITECTURE})
  SET(IS_ARM TRUE)
ENDIF()

SET(BUILD_TEST FALSE)
add_definitions(-std=c++14)

#######################################
SET(PROJECT_NAME fast)
PROJECT(${PROJECT_NAME})
SET(CMAKE_VERBOSE_MAKEFILE ON)

# Build type and flags
SET(CMAKE_BUILD_TYPE Release) # Options:  Debug, RelWithDebInfo, Release
SET(CMAKE_CXX_FLAGS "-Wall -Werror -Wno-unused-variable -Wno-unused-but-set-variable -Wno-unknown-pragmas")
SET(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS} -O0 -g")
IF(IS_ARM)
  SET(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS} -O3 -march=armv8-a")
ELSE()
  SET(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS} -O3 -mmmx -msse -msse -msse2 -msse3 -mssse3 -fomit-frame-pointer")
ENDIF()

# Add Sources
SET(SOURCES  ${PROJECT_SOURCE_DIR}/src/fast_10.cpp
             ${PROJECT_SOURCE_DIR}/src/fast_10_score.cpp
             ${PROJECT_SOURCE_DIR}/src/nonmax_3x3.cpp)
IF(IS_ARM)
#  LIST(APPEND SOURCES ${PROJECT_SOURCE_DIR}/src/faster_corner_9_neon.cpp)
ELSE()
  LIST(APPEND SOURCES ${PROJECT_SOURCE_DIR}/src/faster_corner_10_sse.cpp)
ENDIF()

# Add library
INCLUDE_DIRECTORIES(include src)
ADD_LIBRARY(${PROJECT_NAME} SHARED ${SOURCES})
MESSAGE("PROJECT_NAME set to " ${PROJECT_NAME})

# Add Tests
IF(BUILD_TEST)
  MESSAGE("Building Fast Test")
  FIND_PACKAGE(OpenCV REQUIRED)
  ADD_DEFINITIONS(-DTEST_DATA_DIR=\"${PROJECT_SOURCE_DIR}/test/data\")
  ADD_EXECUTABLE(fast_test test/test.cpp)
  TARGET_LINK_LIBRARIES(fast_test ${OpenCV_LIBS} ${PROJECT_NAME})
ENDIF()

install(TARGETS ${PROJECT_NAME}
        EXPORT fastTargets
        LIBRARY DESTINATION lib
        INCLUDES DESTINATION include)


install(EXPORT fastTargets
      FILE fastTargets.cmake
      DESTINATION  "${CMAKE_INSTALL_PREFIX}/lib/cmake/${PROJECT_NAME}")

configure_file(fastConfig.cmake.in fastConfig.cmake @ONLY)
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/fastConfig.cmake"
    DESTINATION "${CMAKE_INSTALL_PREFIX}/lib/cmake/${PROJECT_NAME}")
