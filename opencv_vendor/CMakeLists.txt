cmake_minimum_required(VERSION 3.8)
project(opencv_vendor)

# Default to C17
if(NOT CMAKE_C_STANDARD)
  set(CMAKE_C_STANDARD 17)
endif()

# Default to C++20
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 20)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# find dependencies
find_package(ament_cmake REQUIRED)

# Check if we already have opencv installed in the system
find_package(OpenCV)
# If opencv is already installed systemwide, we don't need to build it
if (${OpenCV_FOUND})
  message(WARN "Found OpenCV installation: ${OpenCV_DIR}. Using that instead of building it from source.")
  ament_package()
  return()
endif()

# Try to find OpenMP (for multithreading in the algorithms)
include (FindOpenMP) # — since cmake version 2.6.3

# If we have OpenMP, we can use it
if (OPENMP_FOUND)
  set (HAVE_OPENMP TRUE)
  set (CMAKE_CXX_FLAGS “${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}”)
  set (CMAKE_C_FLAGS “${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}”)
else()
  set (HAVE_OPENMP FALSE)
endif()

# If we have Qt, we can use it
find_package(Qt5 COMPONENTS Core Gui Widgets Test Concurrent OpenGL)

if (Qt5_FOUND)
  set (HAVE_QT TRUE)
else()
  set (HAVE_QT FALSE)
endif()

# Required packages for compiling and linking against opencv
find_package(Eigen3 REQUIRED)

# Set the thread library, otherwise VTK will cry
#set(CMAKE_THREAD_LIBS_INIT "-lpthread")
#set(CMAKE_HAVE_THREADS_LIBRARY 1)
#set(CMAKE_USE_WIN32_THREADS_INIT 0)
#set(CMAKE_USE_PTHREADS_INIT 1)
#set(THREADS_PREFER_PTHREAD_FLAG ON)

#find_package(VTK REQUIRED)
#find_package(Iconv REQUIRED)

# Note, if opencv needs fixing, we need to include the headers and link against
# vtk, eigen, thread and Iconv which are all dependencies of opencv
# and are not automatically included by opencv because they are usually at the same level
# as opencv.h and opencv.so files (but in this case, with the manual installation, they are not)

# Set the destination directory for the opencv installation
set(opencv_DESTDIR ${CMAKE_CURRENT_BINARY_DIR}/opencv_install)
include(ExternalProject)
set(opencv_version 4.7.0)

# Clone the opencv_contrib repo
message(STATUS "Cloning OpenCV contrib.")
ExternalProject_Add(opencv_contrib
  GIT_REPOSITORY git@github.com:opencv/opencv_contrib.git
  GIT_TAG ${opencv_version}
  GIT_SHALLOW TRUE
  BUILD_IN_SOURCE FALSE
  SOURCE_DIR ${CMAKE_BINARY_DIR}/opencv-prefix/opencv_contrib
  BUILD_COMMAND ""
  INSTALL_COMMAND ""
  CONFIGURE_COMMAND ""
)

include(ProcessorCount)
ProcessorCount(N)

# Clone and compile the opencv repo
message(STATUS "Cloning and building OpenCV from source.")
ExternalProject_Add(opencv
  GIT_REPOSITORY https://github.com/opencv/opencv
  GIT_TAG ${opencv_version}
  GIT_SHALLOW TRUE
  BUILD_IN_SOURCE FALSE
  SOURCE_DIR ${CMAKE_BINARY_DIR}/opencv-prefix/src/opencv
  BINARY_DIR ${CMAKE_BINARY_DIR}/opencv-prefix/build
  INSTALL_DIR ${opencv_DESTDIR}
  CONFIGURE_COMMAND ${CMAKE_COMMAND} -DCMAKE_INSTALL_PREFIX=<INSTALL_DIR> 
    -DCMAKE_BUILD_TYPE=RELEASE
    -DBUILD_SHARED_LIBS=OFF
    -DWITH_OPENGL=ON 
    -DWITH_QT=ON  
    -DBUILD_JAVA=OFF
    -DBUILD_opencv_python2=OFF
    -DBUILD_opencv_python3=ON
    -DWITH_OPENMP=${HAVE_OPENMP}
    -DWITH_QT=${HAVE_QT}
    -DFORCE_VTK=OFF
    -DWITH_CUDA=OFF
    -DOPENCV_DNN_CUDA=OFF
    -DENABLE_FAST_MATH=ON
    -DCUDA_FAST_MATH=OFF
    -DINSTALL_C_EXAMPLES=OFF
    -DINSTALL_PYTHON_EXAMPLES=OFF
    -DINSTALL_PYTHON_EXAMPLES=OFF
    -DBUILD_EXAMPLES=OFF 
    -DBUILD_DOCS=OFF
    -DBUILD_TESTS=OFF
    -DBUILD_PERF_TESTS=OFF
    -DBUILD_PERF_TESTS=OFF
    -DOPENCV_DOWNLOAD_PATH=/tmp/opencv-cache
    -DOPENCV_EXTRA_MODULES_PATH=${CMAKE_BINARY_DIR}/opencv-prefix/opencv_contrib/modules
    -DWITH_THREAD=ON 
    -DWITH_GSTREAMER=ON
    -DCMAKE_POSITION_INDEPENDENT_CODE=ON
    -DCMAKE_VERBOSE_MAKEFILE=FALSE <SOURCE_DIR>
  BUILD_COMMAND        make -j ${N}
  INSTALL_COMMAND      make install
)

file(GLOB SHARED_LIBS
  "${opencv_DESTDIR}/lib/*.so"
  "${opencv_DESTDIR}/lib/*.a"
)

set(SHORT_LIBS "")

foreach(SHARED_LIB ${SHARED_LIBS})
  string(REPLACE ${opencv_DESTDIR}/lib/ "" SHORT_LIB ${SHARED_LIB})
  list(APPEND SHORT_LIBS ${SHORT_LIB})
endforeach()

ament_export_include_directories(include)
ament_export_libraries(${SHORT_LIBS})

# Move the cmake configurations such that opencv can be found automatically
install(DIRECTORY ${opencv_DESTDIR}/lib/cmake/opencv4/ 
  DESTINATION share/${PROJECT_NAME}/cmake
)

install(DIRECTORY 
  ${opencv_DESTDIR}/include/
  DESTINATION include
)

install(DIRECTORY ${opencv_DESTDIR}/lib/
  DESTINATION lib
)

ament_package()
