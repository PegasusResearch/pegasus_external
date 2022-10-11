cmake_minimum_required(VERSION 3.5)
project(dji_vendor)

# Default to C99
if(NOT CMAKE_C_STANDARD)
  set(CMAKE_C_STANDARD 99)
endif()

# Default to C++20
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 20)
endif()

# Add compilation flags
if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# find dependencies
find_package(ament_cmake REQUIRED)

set(dji_DESTDIR ${CMAKE_CURRENT_BINARY_DIR}/dji_install)

include(ExternalProject)
ExternalProject_Add(dji
  GIT_REPOSITORY https://github.com/dji-sdk/Onboard-SDK.git
  GIT_TAG 3.9.0
  GIT_SHALLOW TRUE
  BUILD_IN_SOURCE TRUE
  INSTALL_DIR ${dji_DESTDIR}
  CONFIGURE_COMMAND ${CMAKE_COMMAND} -DCMAKE_INSTALL_PREFIX=<INSTALL_DIR> -DADVANCED_SENSING=ON
)

#ament_export_include_directories(include)
#ament_export_libraries(${SHORT_LIBS})

#install(DIRECTORY ${dji_DESTDIR}/include/
#  DESTINATION include
#)

#install(DIRECTORY ${dji_DESTDIR}/lib/
#  DESTINATION lib
#)

ament_package()