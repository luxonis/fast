cmake_minimum_required (VERSION 3.4)
#######################################
# user config

# Based on the ARM_ARCHITECTURE environment variable
# set IS_ARM to build on the Odroid board
message("ARM_ARCHITECTURE environment variable set to " ${CMAKE_SYSTEM_PROCESSOR})

if(${CMAKE_SYSTEM_PROCESSOR} MATCHES "arm")
    set(IS_ARM TRUE)
    # set any ARM-specific build flags here.
else()
    message(STATUS "Building on non-ARM architecture")
    set(IS_ARM FALSE)
endif()

if(DEFINED ENV{ARM_ARCHITECTURE})
  message("ARM_ARCHITECTURE environment variable set to " $ENV{ARM_ARCHITECTURE})
  set(IS_ARM TRUE)
endif()

set(BUILD_TEST FALSE)
add_definitions(-std=c++14)

#######################################
set(PROJECT_NAME fast)
project(${PROJECT_NAME} VERSION 0.1.0 LANGUAGES CXX)
# set(CMAKE_VERBOSE_MAKEFILE ON)

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)
include(CMakeDependentOption)

# Build type and flags
# TODO - reapply these flags if need may be
# set(CMAKE_CXX_FLAGS "-Wall -Werror -Wno-unused-variable -Wno-unused-but-set-variable -Wno-unknown-pragmas")
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS} -O0 -g")
if(IS_ARM)
  set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS} -O3 -march=armv8-a")
else()
  # TODO - condition out these flags and the sse impl
  set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS} -O3 -mmmx -msse -msse -msse2 -msse3 -mssse3 -fomit-frame-pointer")
endif()

# Add Sources
set(SOURCES  ${PROJECT_SOURCE_DIR}/src/fast_10.cpp
             ${PROJECT_SOURCE_DIR}/src/fast_10_score.cpp
             ${PROJECT_SOURCE_DIR}/src/nonmax_3x3.cpp)
if(IS_ARM)
#  list(APPEND SOURCES ${PROJECT_SOURCE_DIR}/src/faster_corner_9_neon.cpp)
else()
  list(APPEND SOURCES ${PROJECT_SOURCE_DIR}/src/faster_corner_10_sse.cpp)
endif()

# Add library
include_directories(include src)
add_library(${PROJECT_NAME} SHARED ${SOURCES})
message("PROJECT_NAME set to " ${PROJECT_NAME})
message("PROJECT_VERSION set to " ${PROJECT_VERSION})

set(CMAKEPACKAGE_INSTALL_DIR
    "${CMAKE_INSTALL_DATADIR}/${PROJECT_NAME}/cmake"
    CACHE PATH "The directory relative to CMAKE_INSTALL_PREFIX where ${PACKAGE_NAME}Config.cmake is installed"
    )

target_include_directories(${PROJECT_NAME} PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)

# ---------------------------- new
install(TARGETS ${PROJECT_NAME}
    EXPORT ${PROJECT_NAME}Targets
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    RUNTIME DESTINATION bin
    INCLUDES DESTINATION include
)

install(DIRECTORY include/
    DESTINATION include)

configure_package_config_file (
  ${CMAKE_CURRENT_SOURCE_DIR}/${PROJECT_NAME}Config.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
  INSTALL_DESTINATION ${CMAKEPACKAGE_INSTALL_DIR}
)

# Export the library for use in other projects
install(EXPORT ${PROJECT_NAME}Targets
    NAMESPACE ${PROJECT_NAME}::
    DESTINATION ${CMAKEPACKAGE_INSTALL_DIR})

# Optionally, create a ConfigVersion file
write_basic_package_version_file(${PROJECT_NAME}ConfigVersion.cmake
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY AnyNewerVersion
)

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
         DESTINATION ${CMAKEPACKAGE_INSTALL_DIR})

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
    DESTINATION ${CMAKEPACKAGE_INSTALL_DIR})

export (TARGETS ${PROJECT_NAME} NAMESPACE ${PROJECT_NAME}:: FILE ${PROJECT_NAME}Targets.cmake)
export (PACKAGE ${PROJECT_NAME})
# -------------------- new end
# install(TARGETS ${PROJECT_NAME}
#         EXPORT fastTargets
#         LIBRARY DESTINATION lib
#         INCLUDES DESTINATION include)


# install(EXPORT fastTargets
#       FILE fastTargets.cmake
#       DESTINATION  "${CMAKE_INSTALL_PREFIX}/lib/cmake/${PROJECT_NAME}")

# configure_file(fastConfig.cmake.in fastConfig.cmake @ONLY)
# install(FILES "${CMAKE_CURRENT_BINARY_DIR}/fastConfig.cmake"
#     DESTINATION "${CMAKE_INSTALL_PREFIX}/lib/cmake/${PROJECT_NAME}")

    # Add Tests
if(BUILD_TEST)
  message("Building Fast Test")
  find_package(OpenCV REQUIRED)
  add_definitions(-DTEST_DATA_DIR=\"${PROJECT_SOURCE_DIR}/test/data\")
  add_executable(fast_test test/test.cpp)
  target_link_libraries(fast_test ${OpenCV_LIBS} ${PROJECT_NAME})
endif()