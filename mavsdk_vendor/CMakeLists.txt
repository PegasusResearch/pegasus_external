cmake_minimum_required(VERSION 3.5)
project(mavsdk_vendor)

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

set(mavsdk_DESTDIR ${CMAKE_CURRENT_BINARY_DIR}/mavsdk_install)

include(ProcessorCount)
ProcessorCount(N)

include(ExternalProject)
ExternalProject_Add(mavsdk
  GIT_REPOSITORY https://github.com/mavlink/MAVSDK
  GIT_TAG v2.0.1
  GIT_SHALLOW TRUE
  BUILD_IN_SOURCE TRUE
  INSTALL_DIR ${mavsdk_DESTDIR}
  CONFIGURE_COMMAND ${CMAKE_COMMAND} 
    -DCMAKE_INSTALL_PREFIX=<INSTALL_DIR> 
    -DCMAKE_POSITION_INDEPENDENT_CODE=ON
  BUILD_COMMAND        make -j ${N}
  INSTALL_COMMAND      make install
)

file(GLOB SHARED_LIBS
  "${mavsdk_DESTDIR}/lib/*.so"
)

set(SHORT_LIBS "")

foreach(SHARED_LIB ${SHARED_LIBS})
  string(REPLACE ${mavsdk_DESTDIR}/lib/ "" SHORT_LIB ${SHARED_LIB})
  list(APPEND SHORT_LIBS ${SHORT_LIB})
endforeach()

ament_export_include_directories(include)
ament_export_libraries(${SHORT_LIBS})

# Move the cmake configurations such that mavsdk can be found automatically
install(DIRECTORY ${mavsdk_DESTDIR}/lib/cmake/MAVSDK/ 
  DESTINATION share/${PROJECT_NAME}/cmake
)

install(DIRECTORY ${mavsdk_DESTDIR}/include/
  DESTINATION include
)

install(DIRECTORY ${mavsdk_DESTDIR}/lib/
  DESTINATION lib
)

ament_package()
