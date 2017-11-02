# ----------------------------------------------------------------------------
#  CMake file for libturbojpeg. See root CMakeLists.txt
#
# ----------------------------------------------------------------------------
project(${JPEG_LIBRARY})

ocv_include_directories(${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_BINARY_DIR})

set(LIB_TURBOJPEG_SOURCES 
  jaricom.c jcarith.c jdarith.c
  jcapimin.c jcapistd.c jccoefct.c jccolor.c 
  jcdctmgr.c jchuff.c jcinit.c jcmainct.c jcmarker.c jcmaster.c 
  jcomapi.c jcparam.c jcphuff.c jcprepct.c jcsample.c jctrans.c 
  jdapimin.c jdapistd.c jdatadst.c jdatasrc.c jdcoefct.c jdcolor.c 
  jddctmgr.c jdhuff.c jdinput.c jdmainct.c jdmarker.c jdmaster.c 
  jdmerge.c jdphuff.c jdpostct.c jdsample.c jdtrans.c jerror.c 
  jfdctflt.c jfdctfst.c jfdctint.c jidctflt.c jidctfst.c jidctint.c 
  jidctred.c jquant1.c jquant2.c jutils.c jmemmgr.c jmemnobs.c
  turbojpeg.c turbojpeg.h transupp.c transupp.h jdatadst-tj.c jdatasrc-tj.c)

if(ARM)
  if("${CMAKE_OSX_ARCHITECTURES}" STREQUAL "arm64")
    set(LIB_TURBOJPEG_SOURCES ${LIB_TURBOJPEG_SOURCES} simd/jsimd_arm64.c simd/jsimd_arm64_neon.S)
  else()
    set(LIB_TURBOJPEG_SOURCES ${LIB_TURBOJPEG_SOURCES} simd/jsimd_arm.c simd/jsimd_arm_neon.S)
  endif()
else()
  set(LIB_TURBOJPEG_SOURCES ${LIB_TURBOJPEG_SOURCES} jsimd_none.c)
endif()

math(EXPR SIZEOF_SIZE_T "${SIZEOF_VOID_P_BITS}/8")
add_definitions(-DSIZEOF_SIZE_T=${SIZEOF_SIZE_T})

# ----------------------------------------------------------------------------------
#         Define the library target:
# ----------------------------------------------------------------------------------

add_library(${JPEG_LIBRARY} STATIC ${LIB_TURBOJPEG_SOURCES})

if(UNIX)
  if(CMAKE_COMPILER_IS_GNUCXX OR CV_ICC)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fPIC")
  endif()
endif()

if(CMAKE_COMPILER_IS_GNUCXX)
  set_source_files_properties(jcdctmgr.c PROPERTIES COMPILE_FLAGS "-O1")
endif()

ocv_warnings_disable(CMAKE_C_FLAGS -Wcast-align -Wshadow -Wunused)
ocv_warnings_disable(CMAKE_C_FLAGS -Wunused-parameter) # clang
ocv_warnings_disable(CMAKE_C_FLAGS /wd4013 /wd4244 /wd4267) # vs2005

set_target_properties(${JPEG_LIBRARY}
  PROPERTIES OUTPUT_NAME ${JPEG_LIBRARY}
  DEBUG_POSTFIX "${OPENCV_DEBUG_POSTFIX}"
  COMPILE_PDB_NAME ${JPEG_LIBRARY}
  COMPILE_PDB_NAME_DEBUG "${JPEG_LIBRARY}${OPENCV_DEBUG_POSTFIX}"
  ARCHIVE_OUTPUT_DIRECTORY ${3P_LIBRARY_OUTPUT_PATH})

if(ENABLE_SOLUTION_FOLDERS)
  set_target_properties(${JPEG_LIBRARY} PROPERTIES FOLDER "3rdparty")
endif()

if(NOT BUILD_SHARED_LIBS)
  ocv_install_target(${JPEG_LIBRARY} EXPORT OpenCVModules ARCHIVE DESTINATION ${OPENCV_3P_LIB_INSTALL_PATH} COMPONENT dev)
endif()

install(FILES jconfig.h jerror.h jmorecfg.h jpeglib.h turbojpeg.h DESTINATION "${OPENCV_INCLUDE_INSTALL_PATH}/opencv2/${JPEG_LIBRARY}/")

set(OPENCV_MODULES_PUBLIC ${OPENCV_MODULES_PUBLIC} "${JPEG_LIBRARY}" CACHE INTERNAL "List of OpenCV modules marked for export")
