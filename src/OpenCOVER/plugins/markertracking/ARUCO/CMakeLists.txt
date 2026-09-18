if(NOT COVISE_USE_OPENCV4 AND NOT COVISE_USE_OPENCV5)
    message(WARNING
      "ARUCO plugin disabled: OpenCV not found or unsupported. "
      "Supported versions are OpenCV 4 and 5.")
    return()
endif()
USING(OpenCV)

# Resolve calibration lib robustly:
# OpenCV4: opencv_calib3d
# Some OpenCV5 packages: opencv_calib
set(ARUCO_CALIB_LIB "")
if(DEFINED OPENCV_CALIB3D_LIBRARY AND NOT OPENCV_CALIB3D_LIBRARY STREQUAL "OPENCV_CALIB3D_LIBRARY-NOTFOUND")
    set(ARUCO_CALIB_LIB "${OPENCV_CALIB3D_LIBRARY}")
elseif(DEFINED OPENCV_CALIB_LIBRARY AND NOT OPENCV_CALIB_LIBRARY STREQUAL "OPENCV_CALIB_LIBRARY-NOTFOUND")
    set(ARUCO_CALIB_LIB "${OPENCV_CALIB_LIBRARY}")
else()
    find_library(ARUCO_CALIB_LIB
        NAMES opencv_calib3d opencv_calib
        PATHS /usr /usr/local
        PATH_SUFFIXES lib lib64
    )
endif()

if(NOT ARUCO_CALIB_LIB)
    message(FATAL_ERROR "ARUCO: no OpenCV calibration library found (opencv_calib3d/opencv_calib)")
endif()

message(STATUS "ARUCO calib lib: ${ARUCO_CALIB_LIB}")

# Resolve geometry lib (OpenCV5 package split)
set(ARUCO_GEOMETRY_LIB "")

if(DEFINED OPENCV_GEOMETRY_LIBRARY
   AND NOT OPENCV_GEOMETRY_LIBRARY STREQUAL ""
   AND NOT OPENCV_GEOMETRY_LIBRARY STREQUAL "OPENCV_GEOMETRY_LIBRARY-NOTFOUND"
   AND EXISTS "${OPENCV_GEOMETRY_LIBRARY}")
    set(ARUCO_GEOMETRY_LIB "${OPENCV_GEOMETRY_LIBRARY}")
else()
    find_library(ARUCO_GEOMETRY_LIB
        NAMES opencv_geometry
        PATHS /usr /usr/local
        PATH_SUFFIXES lib lib64
    )

    if(NOT ARUCO_GEOMETRY_LIB OR ARUCO_GEOMETRY_LIB STREQUAL "ARUCO_GEOMETRY_LIB-NOTFOUND")
        find_file(ARUCO_GEOMETRY_LIB
            NAMES libopencv_geometry.so libopencv_geometry.so.500 libopencv_geometry.so.5.0.0
            PATHS /usr/lib /usr/local/lib /usr/lib64 /usr/local/lib64
        )
    endif()
endif()

message(STATUS "ARUCO geometry lib: ${ARUCO_GEOMETRY_LIB}")

set(ARUCO_OPENCV_LIBS
    ${OPENCV_CORE_LIBRARY}
    ${OPENCV_IMGPROC_LIBRARY}
    ${OPENCV_VIDEOIO_LIBRARY}
    ${OPENCV_HIGHGUI_LIBRARY}
    ${OPENCV_OBJDETECT_LIBRARY}
    ${ARUCO_CALIB_LIB}
    ${ARUCO_GEOMETRY_LIB}
)
list(FILTER ARUCO_OPENCV_LIBS EXCLUDE REGEX ".*-NOTFOUND$")
message(STATUS "ARUCO OpenCV libs: ${ARUCO_OPENCV_LIBS}")

SET(ARUCO_USE_CUDA FALSE)

IF(ARUCO_USE_CUDA)

IF(CUDA_FOUND)
# Example of how to find an include file from the CUDA_SDK_ROOT_DIR

find_path(CUDA_CUT_INCLUDE_DIR
  cutil.h
  PATHS ${CUDA_SDK_SEARCH_PATH}
  PATH_SUFFIXES "common/inc"
  DOC "Location of cutil.h"
  NO_DEFAULT_PATH
  )
# Now search system paths
find_path(CUDA_CUT_INCLUDE_DIR cutil.h DOC "Location of cutil.h")

mark_as_advanced(CUDA_CUT_INCLUDE_DIR)


# cutil library is called cutil64 for 64 bit builds on windows.  We don't want
# to get these confused, so we are setting the name based on the word size of
# the build.

IF(MSVC)
if(CMAKE_SIZEOF_VOID_P EQUAL 8)
  set(cuda_cutil_name cutil64 libcutil_x86_64.a cutil libcutil.so)
else(CMAKE_SIZEOF_VOID_P EQUAL 8)
  set(cuda_cutil_name cutil32)
endif(CMAKE_SIZEOF_VOID_P EQUAL 8)

set(CUDA_SDK_SEARCH_PATH ${CUDA_SDK_SEARCH_PATH} $ENV{EXTERNLIBS}/cuda)
find_library(CUDA_CUT_LIBRARY
  NAMES cutil ${cuda_cutil_name}
  PATHS ${CUDA_SDK_SEARCH_PATH}
  # The new version of the sdk shows up in common/lib, but the old one is in lib
  PATH_SUFFIXES "common/lib" "common/lib/x64" "lib" "C/lib"
  DOC "Location of cutil library"
  NO_DEFAULT_PATH
 )
# Now search system paths
find_library(CUDA_CUT_LIBRARY NAMES cutil ${cuda_cutil_name} DOC "Location of cutil library")

if(CMAKE_SIZEOF_VOID_P EQUAL 8)
  set(cuda_cutild_name cutil64D libcutil_x86_64.a cutil libcutil.so)
else(CMAKE_SIZEOF_VOID_P EQUAL 8)
  set(cuda_cutild_name cutil32D)
endif(CMAKE_SIZEOF_VOID_P EQUAL 8)

find_library(CUDA_CUT_LIBRARYD
  NAMES cutilD ${cuda_cutild_name}
  PATHS ${CUDA_SDK_SEARCH_PATH}
  # The new version of the sdk shows up in common/lib, but the old one is in lib
  PATH_SUFFIXES "common/lib" "common/lib/x64" "lib" "C/lib"
  DOC "Location of cutil Debug library"
  NO_DEFAULT_PATH
 )
# Now search system paths
find_library(CUDA_CUT_LIBRARYD NAMES cutild ${cuda_cutild_name} DOC "Location of cutil library")

    SET(CUDA_CUT_LIBRARIES optimized ${CUDA_CUT_LIBRARY} debug ${CUDA_CUT_LIBRARYD})
ELSE(MSVC)
if(CMAKE_SIZEOF_VOID_P EQUAL 8)
  set(cuda_cutil_name cutil64 libcutil_x86_64.a cutil libcutil.so)
else(CMAKE_SIZEOF_VOID_P EQUAL 8)
  set(cuda_cutil_name cutil32)
endif(CMAKE_SIZEOF_VOID_P EQUAL 8)

set(CUDA_SDK_SEARCH_PATH ${CUDA_SDK_SEARCH_PATH} $ENV{EXTERNLIBS}/cuda)
find_library(CUDA_CUT_LIBRARY
  NAMES cutil ${cuda_cutil_name}
  PATHS ${CUDA_SDK_SEARCH_PATH}
  # The new version of the sdk shows up in common/lib, but the old one is in lib
  PATH_SUFFIXES "common/lib" "common/lib/x64" "lib" "C/lib"
  DOC "Location of cutil library"
  NO_DEFAULT_PATH
 )
# Now search system paths
find_library(CUDA_CUT_LIBRARY NAMES cutil ${cuda_cutil_name} DOC "Location of cutil library")
mark_as_advanced(CUDA_CUT_LIBRARY)
set(CUDA_CUT_LIBRARIES ${CUDA_CUT_LIBRARY})
ENDIF(MSVC)
ENDIF(CUDA_FOUND)

ENDIF(ARUCO_USE_CUDA)


#INCLUDE_DIRECTORIES(${XERCESC_INCLUDE_DIR})

SET(HEADERS
  ARUCOTrackerPlugin.h
  Marker.h
  MatrixUtil.h
#  CvCapture.h
  ../common/RemoteAR.h
)

SET(SOURCES
  ARUCOTrackerPlugin.cpp
  Marker.cpp
  MatrixUtil.cpp
#  CvCapture.cpp
  ../common/RemoteAR.cpp
)


IF(ARUCO_USE_CUDA)
IF(CUDA_FOUND)
  cover_add_plugin(ARUCO ${CUDA_CUT_LIBRARIES} ${HEADERS} ${SOURCES})
ELSE(CUDA_FOUND)
  cover_add_plugin(ARUCO ${HEADERS} ${SOURCES})
ENDIF(CUDA_FOUND)
ELSE(ARUCO_USE_CUDA)
  cover_add_plugin(ARUCO ${HEADERS} ${SOURCES})
ENDIF(ARUCO_USE_CUDA)

# after cover_add_plugin(ARUCO ...)
find_library(ARUCO_GEOMETRY_LIB NAMES opencv_geometry REQUIRED PATHS /usr/lib /usr/local/lib)
find_library(ARUCO_CALIB_LIB    NAMES opencv_calib3d opencv_calib REQUIRED PATHS /usr/lib /usr/local/lib)

message(STATUS "ARUCO_GEOMETRY_LIB=${ARUCO_GEOMETRY_LIB}")
message(STATUS "ARUCO_CALIB_LIB=${ARUCO_CALIB_LIB}")

if(TARGET ARUCO)
  target_link_libraries(ARUCO
    /usr/lib/libopencv_core.so
    /usr/lib/libopencv_imgproc.so
    /usr/lib/libopencv_videoio.so
    /usr/lib/libopencv_highgui.so
    /usr/lib/libopencv_objdetect.so
    /usr/lib/libopencv_calib.so
    /usr/lib/libopencv_geometry.so
  )
endif()

SET(HEADERS
)
SET(SOURCES
  create_marker.cpp
)

ADD_COVISE_EXECUTABLE(createMarker)
COVISE_WNOERROR(createMarker)

TARGET_LINK_LIBRARIES(createMarker ${EXTRA_LIBS} )

