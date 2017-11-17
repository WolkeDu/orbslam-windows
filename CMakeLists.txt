cmake_minimum_required(VERSION 2.8)

set(CMAKE_USER_MAKE_RULES_OVERRIDE
   ${CMAKE_CURRENT_SOURCE_DIR}/compiler_flags_overrides.cmake)

project(ORB_SLAM2)
message(${CMAKE_CURRENT_SOURCE_DIR})
SET(ORBSLAM2_LIB_TYPE SHARED)
option( BUILD_EXAMPLES "Build Examples" ON )
option( ORBSLAM2_STATIC_LIB "Orb-slam lib type" OFF )
 
IF(NOT CMAKE_BUILD_TYPE)
  SET(CMAKE_BUILD_TYPE Release)
ENDIF()

if(ORBSLAM2_STATIC_LIB)
  SET(ORBSLAM2_LIB_TYPE STATIC)
endif()

MESSAGE("Build type: " ${CMAKE_BUILD_TYPE})

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}  -W0 -MT -bigobj")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -W0 -MT -bigobj")

# Check C++11 or C++0x support
include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
SET(DBUILD_SHARED true)

if(COMPILER_SUPPORTS_CXX11)
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
   add_definitions(-DCOMPILEDWITHC11)
   message(STATUS "Using flag -std=c++11.")
elseif(COMPILER_SUPPORTS_CXX0X)
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
   add_definitions(-DCOMPILEDWITHC0X)
   message(STATUS "Using flag -std=c++0x.")
#else()
#   message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif()

LIST(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake_modules)

IF(WIN32)
    option( BUILD_SHARED_LIBS    "Build Shared Library" OFF)
#    option( MSVC_USE_STATIC_CRT  "Use static C Runtime with MSVC, /MT instead of /MD" ON)
 
	add_definitions("-DNOMINMAX")
	add_definitions("-DWINDOWS")
	add_definitions("-DORBSLAM_EXPORT")
	add_definitions(-DCOMPILEDWITHC11)
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -bigobj")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -bigobj")

else()
    option( BUILD_SHARED_LIBS "Build Shared Library" ON)
endif()

find_package(OpenCV 3.0 QUIET)
if(NOT OpenCV_FOUND)
   find_package(OpenCV 2.4.3 QUIET)
   if(NOT OpenCV_FOUND)
      message(FATAL_ERROR "OpenCV > 2.4.3 not found.")
   endif()
endif()

find_package(Eigen3 3.1.0 REQUIRED)
find_package(Pangolin REQUIRED)



include_directories(
${PROJECT_SOURCE_DIR}
${PROJECT_SOURCE_DIR}/include
${EIGEN3_INCLUDE_DIR}
${Pangolin_INCLUDE_DIRS}
)

#add thirdParty lib
add_subdirectory(Thirdparty/DBoW2)
add_subdirectory(Thirdparty/g2o)

set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/lib)

IF(WIN32)
	set(ORB_SLAM_G2O_LIBRARIES ${PROJECT_SOURCE_DIR}/Thirdparty/g2o/build/${CMAKE_BUILD_TYPE}/g2o.lib)
	set(ORB_SLAM_DBoW2_LIBRARIES ${PROJECT_SOURCE_DIR}/Thirdparty/DBoW2/bin/${CMAKE_BUILD_TYPE}/DBoW2.lib)
	set(ORB_THIRD_PARTY_LIB ${ORB_SLAM_G2O_LIBRARIES} ${ORB_SLAM_DBoW2_LIBRARIES})
else()
  set(ORB_SLAM_G2O_LIBRARIES ${PROJECT_SOURCE_DIR}/Thirdparty/DBoW2/lib/libDBoW2.so)
  set(ORB_SLAM_DBoW2_LIBRARIES ${PROJECT_SOURCE_DIR}/Thirdparty/g2o/lib/libg2o.so)
ENDIF()

add_library(${PROJECT_NAME} ${ORBSLAM2_LIB_TYPE}
src/System.cc
src/Tracking.cc
src/LocalMapping.cc
src/LoopClosing.cc
src/ORBextractor.cc
src/ORBmatcher.cc
src/FrameDrawer.cc
src/Converter.cc
src/MapPoint.cc
src/KeyFrame.cc
src/Map.cc
src/MapDrawer.cc
src/Optimizer.cc
src/PnPsolver.cc
src/Frame.cc
src/KeyFrameDatabase.cc
src/Sim3Solver.cc
src/Initializer.cc
src/Viewer.cc

)

target_link_libraries(${PROJECT_NAME}
${OpenCV_LIBS}
${EIGEN3_LIBS}
${Pangolin_LIBRARIES}
${ORB_SLAM_G2O_LIBRARIES}
${ORB_SLAM_DBoW2_LIBRARIES}
#${PROJECT_SOURCE_DIR}/Thirdparty/DBoW2/bin/Release/DBoW2.lib
#${PROJECT_SOURCE_DIR}/Thirdparty/g2o/build/Release/g2o.lib
)

# Build examples
if(BUILD_EXAMPLES)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/Examples/RGB-D)

add_executable(rgbd_tum
Examples/RGB-D/rgbd_tum.cc)
target_link_libraries(rgbd_tum ${PROJECT_NAME} ${ORB_THIRD_PARTY_LIB})

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/Examples/Stereo)

add_executable(stereo_kitti
Examples/Stereo/stereo_kitti.cc)
target_link_libraries(stereo_kitti ${PROJECT_NAME} ${ORB_THIRD_PARTY_LIB})

add_executable(stereo_euroc
Examples/Stereo/stereo_euroc.cc)
target_link_libraries(stereo_euroc ${PROJECT_NAME} ${ORB_THIRD_PARTY_LIB})


set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/Examples/Monocular)

add_executable(mono_tum
Examples/Monocular/mono_tum.cc)
target_link_libraries(mono_tum ${PROJECT_NAME} ${ORB_THIRD_PARTY_LIB})

add_executable(mono_kitti
Examples/Monocular/mono_kitti.cc)
target_link_libraries(mono_kitti ${PROJECT_NAME} ${ORB_THIRD_PARTY_LIB})

add_executable(mono_euroc
Examples/Monocular/mono_euroc.cc)
target_link_libraries(mono_euroc ${PROJECT_NAME} ${ORB_THIRD_PARTY_LIB})

ENDIF()

# Build tools
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/tools)
add_executable(bin_vocabulary
tools/bin_vocabulary.cc)
target_link_libraries(bin_vocabulary ${PROJECT_NAME} ${ORB_THIRD_PARTY_LIB})

