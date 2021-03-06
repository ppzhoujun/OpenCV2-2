# ----------------------------------------------------------------------------
#  Root CMake file for Android Opencv Build
#
#    To build with cmake
#        $ mkdir build
#        $ cd build
#        $ cmake ..
#        $ make
#    Make sure to set the path in the cache for the crystax ndk available 
#    here:
#          http://www.crystax.net/android/ndk-r4.php
#
#   - initial version August 2010 Ethan Rublee ethan.rublee@gmail.com
#
# ----------------------------------------------------------------------------

project(android-opencv-shared)

cmake_minimum_required(VERSION 2.8)

set(opencv_root "$ENV{HOME}/opencv" CACHE PATH "opencv source root directory")
if(NOT EXISTS ${opencv_root})
message(FATAL_ERROR "Cannot find your opencv root directory!" )
endif()

set(android_opencv_mk "${opencv_root}/android/build/android-opencv.mk" cache FILE "the generated android-opencv.mk file")
if(NOT EXISTS ${android_opencv_mk})
message(FATAL_ERROR "please locate the cmake generated android-opencv.mk file, usually in the android/build directory...")
endif()

set(ANDROID true)

file(GLOB module_includes "${opencv_root}/modules/[a-zA-Z]*")

list(REMOVE_ITEM module_includes ${opencv_root}/modules/CMakeLists.txt)

set(module_includes ${module_includes} ${CMAKE_SOURCE_DIR}/../3rdparty $(OPENCV_BUILD_ROOT) )
foreach(mdir ${module_includes})
    string(REPLACE "${opencv_root}" "$(OPENCV_ROOT)" n_f ${mdir})	
    set(android_module_include_dirs "${android_module_include_dirs} ${n_f}/include")
endforeach()

configure_file("${CMAKE_SOURCE_DIR}/Android.mk.master.in" "${CMAKE_BINARY_DIR}/Android.mk")
configure_file("${CMAKE_SOURCE_DIR}/Application.mk.in" "${CMAKE_BINARY_DIR}/Application.mk")
configure_file("${CMAKE_SOURCE_DIR}/AndroidManifest.xml.in" "${CMAKE_BINARY_DIR}/AndroidManifest.xml")
configure_file("${CMAKE_SOURCE_DIR}/default.properties.in" "${CMAKE_BINARY_DIR}/default.properties")

set(NDK_ROOT  "$ENV{HOME}/android-ndk-r4-crystax" CACHE STRING "the crystax ndk directory")
if(NOT EXISTS ${NDK_ROOT})
message(FATAL_ERROR "Cannot find your ndk root directory! please download and 
unzip the android ndk from crystax to the directory specified by NDK_ROOT
You may download the crystax ndk from: 
        http://www.crystax.net/android/ndk-r4.php" )
endif()

set(J  "2" CACHE STRING "how many processes for make -j <J>")

ADD_CUSTOM_COMMAND(
OUTPUT    android-ndk
DEPENDS   ${CMAKE_BINARY_DIR}/Android.mk
COMMAND   "${NDK_ROOT}/ndk-build" 
ARGS      --directory=${CMAKE_BINARY_DIR} NDK_APPLICATION_MK=Application.mk -j${J}
)

ADD_CUSTOM_COMMAND(
OUTPUT    android-swig
DEPENDS   ${SWIG_MAIN}
COMMAND   "swig" 
ARGS      --directory=${CMAKE_BINARY_DIR} NDK_APPLICATION_MK=Application.mk -j${J}
)


configure_file("${CMAKE_SOURCE_DIR}/android-opencv.mk.in" "${CMAKE_BINARY_DIR}/android-opencv.mk")

ADD_CUSTOM_TARGET(ndk ALL echo
  DEPENDS android-ndk
)

message(STATUS "Make will use make -j${J} - for speeding up build - you may change this in the cache")
message(STATUS "The NDK directory is ${NDK_ROOT}")
message(STATUS "OpenCV source root is ${opencv_root}")
message(STATUS "just run make - and grab some coffee or tea ;)")
message(STATUS "The android opencv libs will be located in ${CMAKE_BINARY_DIR}/obj/local/armeabi*/")



  



