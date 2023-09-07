cmake_minimum_required(VERSION 3.5)
project(sophus_vendor)

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

set(sophus_DESTDIR ${CMAKE_CURRENT_BINARY_DIR}/sophus_install)

include(ProcessorCount)
ProcessorCount(N)

include(ExternalProject)
ExternalProject_Add(sophus
  GIT_REPOSITORY https://github.com/strasdat/Sophus.git
  GIT_TAG 1.22.10
  GIT_SHALLOW TRUE
  BUILD_IN_SOURCE TRUE
  INSTALL_DIR ${sophus_DESTDIR}
  CONFIGURE_COMMAND ${CMAKE_COMMAND} 
    -DCMAKE_INSTALL_PREFIX=<INSTALL_DIR> 
    -DCMAKE_POSITION_INDEPENDENT_CODE=ON
  BUILD_COMMAND        make -j ${N}
  INSTALL_COMMAND      make install
  TEST_COMMAND         make test
)

ament_export_include_directories(include)

# Move the cmake configurations such that sophus can be found automatically
install(DIRECTORY ${sophus_DESTDIR}/share/sophus/cmake/
  DESTINATION share/${PROJECT_NAME}/cmake
)

install(DIRECTORY ${sophus_DESTDIR}/include/
  DESTINATION include
)

ament_package()
