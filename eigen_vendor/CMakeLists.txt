cmake_minimum_required(VERSION 3.8)
project(eigen_vendor)

# Default to C17
if(NOT CMAKE_C_STANDARD)
  set(CMAKE_C_STANDARD 17)
endif()

# Default to C++20
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 20)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic -O3)
endif()

# find dependencies
find_package(ament_cmake REQUIRED)

# Check if we already have eigen installed in the system
find_package(Eigen3 3.4)
# If opencv is already installed systemwide, we don't need to build it
if (${EIGEN_FOUND})
  message(WARN "Found Eigen installation: ${EIGEN_INCLUDE_DIR}. Using that instead of building it from source.")
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

set(eigen_DESTDIR ${CMAKE_CURRENT_BINARY_DIR}/eigen_install)

include(ProcessorCount)
ProcessorCount(N)

include(ExternalProject)
ExternalProject_Add(eigen
  GIT_REPOSITORY https://gitlab.com/libeigen/eigen.git
  GIT_TAG 3.4.0
  GIT_SHALLOW TRUE
  BUILD_IN_SOURCE FALSE
  SOURCE_DIR ${CMAKE_BINARY_DIR}/eigen-prefix/src
  BINARY_DIR ${CMAKE_BINARY_DIR}/eigen-prefix/build
  INSTALL_DIR ${eigen_DESTDIR}
  CONFIGURE_COMMAND ${CMAKE_COMMAND} 
    -DCMAKE_INSTALL_PREFIX=<INSTALL_DIR> 
    -DCMAKE_POSITION_INDEPENDENT_CODE=ON
    -DEIGEN_TEST_OPENMP=${HAVE_OPENMP}
  BUILD_COMMAND        make -j ${N}
  INSTALL_COMMAND      make install
)

file(GLOB SHARED_LIBS
  "${eigen_DESTDIR}/lib/*.so"
)

set(SHORT_LIBS "")

foreach(SHARED_LIB ${SHARED_LIBS})
  string(REPLACE ${eigen_DESTDIR}/lib/ "" SHORT_LIB ${SHARED_LIB})
  list(APPEND SHORT_LIBS ${SHORT_LIB})
endforeach()

ament_export_include_directories(include)
ament_export_libraries(${SHORT_LIBS})

# Move the cmake configurations such that eigen can be found automatically
install(DIRECTORY ${eigen_DESTDIR}/lib/cmake/eigen/ 
  DESTINATION share/${PROJECT_NAME}/cmake
)

install(DIRECTORY ${eigen_DESTDIR}/include/
  DESTINATION include
)

install(DIRECTORY ${eigen_DESTDIR}/lib/
  DESTINATION lib
)

ament_package()
