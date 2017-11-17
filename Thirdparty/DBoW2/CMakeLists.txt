cmake_minimum_required(VERSION 2.8)
project(DBoW2)

#SET (DBOW2_LIB_TYPE SHARED)
#option( DBOW2_STATIC_LIB "Orb-slam lib type" OFF )
 
#if(DBOW2_STATIC_LIB)
#	SET(DBOW2_LIB_TYPE STATIC)
#endif()
 	
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}  -Wall  -O3 -march=native ")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall  -O3 -march=native")
if(WIN32)
    option( BUILD_SHARED_LIBS    "Build Shared Library" OFF)
    option( MSVC_USE_STATIC_CRT  "Use static C Runtime with MSVC, /MT instead of /MD" ON)
    # Make sure there are no erroneous C Runtime flags

else()
    option( BUILD_SHARED_LIBS "Build Shared Library" ON)
endif()

set(HDRS_DBOW2
  DBoW2/BowVector.h
  DBoW2/FORB.h 
  DBoW2/FClass.h       
  DBoW2/FeatureVector.h
  DBoW2/ScoringObject.h   
  DBoW2/TemplatedVocabulary.h)
set(SRCS_DBOW2
  DBoW2/BowVector.cpp
  DBoW2/FORB.cpp      
  DBoW2/FeatureVector.cpp
  DBoW2/ScoringObject.cpp)

set(HDRS_DUTILS
  DUtils/Random.h
  DUtils/Timestamp.h)
set(SRCS_DUTILS
  DUtils/Random.cpp
  DUtils/Timestamp.cpp)

## Set the output directory for the build executables and libraries
#SET(DBoW2_RUNTIME_OUTPUT_DIRECTORY ${DBoW2_SOURCE_DIR}/bin CACHE PATH "Target for the binaries")
#IF(WIN32)
#  SET(DBoW2_LIBRARY_OUTPUT_DIRECTORY ${DBoW2_SOURCE_DIR}/bin CACHE PATH "Target for the libraries")
#ELSE(WIN32)
#  SET(DBoW2_LIBRARY_OUTPUT_DIRECTORY ${DBoW2_SOURCE_DIR}/lib CACHE PATH "Target for the libraries")
#ENDIF(WIN32)
#SET(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${DBoW2_LIBRARY_OUTPUT_DIRECTORY})
#SET(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${DBoW2_LIBRARY_OUTPUT_DIRECTORY})
#SET(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${DBoW2_RUNTIME_OUTPUT_DIRECTORY})

find_package(OpenCV 3.0 QUIET)
if(NOT OpenCV_FOUND)
   find_package(OpenCV 2.4.3 QUIET)
   if(NOT OpenCV_FOUND)
      message(FATAL_ERROR "OpenCV > 2.4.3 not found.")
   endif()
endif()

SET(DBUILD_SHARED true)
set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)

include_directories(${OpenCV_INCLUDE_DIRS})

add_library(DBoW2 ${DBOW2_LIB_TYPE} ${SRCS_DBOW2} ${SRCS_DUTILS})
#add_library(DBoW2 SHARED ${SRCS_DBOW2} ${SRCS_DUTILS})
#add_library(DBoW2 SHARED ${PROJECT_HEADERS} ${PROJECT_SOURCES})
target_link_libraries(DBoW2 ${OpenCV_LIBS})

