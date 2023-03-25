cmake_minimum_required(VERSION 3.5)
project(vrpn_vendor)

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

set(vrpn_DESTDIR ${CMAKE_CURRENT_BINARY_DIR}/vrpn_install)

include(ExternalProject)
ExternalProject_Add(vrpn
    GIT_REPOSITORY https://github.com/vrpn/vrpn
    GIT_TAG v07.35
    GIT_SHALLOW TRUE
    BUILD_IN_SOURCE FALSE
    SOURCE_DIR           ${CMAKE_BINARY_DIR}/vrpn-prefix/vrpn/src
    BINARY_DIR           ${CMAKE_BINARY_DIR}/vrpn-prefix/vrpn/build
    INSTALL_DIR          ${vrpn_DESTDIR}
    CONFIGURE_COMMAND    ${CMAKE_COMMAND} -DCMAKE_VERBOSE_MAKEFILE=FALSE <SOURCE_DIR>
    BUILD_COMMAND        make
    INSTALL_COMMAND      make install DESTDIR=<INSTALL_DIR>
)

file(GLOB SHARED_LIBS
    "${vrpn_DESTDIR}/usr/local/lib/*.a"
    "${vrpn_DESTDIR}/usr/local/lib/*.so"
)

set(SHORT_LIBS "")

foreach(SHARED_LIB ${SHARED_LIBS})
    string(REPLACE ${vrpn_DESTDIR}/usr/local/lib/ "" SHORT_LIB ${SHARED_LIB})
    list(APPEND SHORT_LIBS ${SHORT_LIB})
endforeach()

ament_export_include_directories(include)
ament_export_libraries(libquat.a libvrpn.a libquat.a)

install(
    DIRECTORY ${vrpn_DESTDIR}/usr/local/include/
    DESTINATION include
)

install(
    DIRECTORY ${vrpn_DESTDIR}/usr/local/lib/
    DESTINATION lib
)

# Create the VRPN libraries manually
# add_library(libvrpn)
# set_target_properties(libvrpn PROPERTIES IMPORTED_LOCATION ${vrpn_DESTDIR}/usr/local/lib/libvrpn.a)
# set_target_properties(libvrpn PROPERTIES INTERFACE_INCLUDE_DIRECTORIES ${vrpn_DESTDIR}/usr/local/include)

# add_library(libquat)
# set_target_properties(libquat PROPERTIES IMPORTED_LOCATION ${vrpn_DESTDIR}/usr/local/lib/libquat.a)
# set_target_properties(libquat PROPERTIES INTERFACE_INCLUDE_DIRECTORIES ${vrpn_DESTDIR}/usr/local/include)

# ament_export_libraries(libvrpn PROPERTIES IMPORTED_LOCATION ${vrpn_DESTDIR}/usr/local/lib/libvrpn.a)
# ament_export_libraries(libquat PROPERTIES IMPORTED_LOCATION ${vrpn_DESTDIR}/usr/local/lib/libquat.a)

ament_package()