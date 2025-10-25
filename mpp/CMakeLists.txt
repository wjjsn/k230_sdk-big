cmake_minimum_required(VERSION 3.16)

# Interface library providing Media Processing Platform headers and static libraries.
add_library(k230_mpp INTERFACE)

set(_K230_MPP_ROOT ${CMAKE_CURRENT_LIST_DIR})
set(_K230_MPP_INCLUDE_DIRS
    ${_K230_MPP_ROOT}/include
    ${_K230_MPP_ROOT}/include/comm
    ${_K230_MPP_ROOT}/include/ioctl
    ${_K230_MPP_ROOT}/userapp/api
)

# Expose the public include directories from the legacy Makefile.
target_include_directories(k230_mpp INTERFACE ${_K230_MPP_INCLUDE_DIRS})

# Collect all static libraries that used to be linked through the Makefile.
file(GLOB _K230_MPP_LIBS CONFIGURE_DEPENDS "${_K230_MPP_ROOT}/userapp/lib/lib*.a")
if(NOT _K230_MPP_LIBS)
    message(WARNING "No static libraries were found under ${_K230_MPP_ROOT}/userapp/lib")
endif()

# Link the collected static libraries into the interface to preserve transitive usage requirements.
target_link_libraries(k230_mpp INTERFACE ${_K230_MPP_LIBS})

# Provide convenient variables for consumers that may still rely on them.
set(K230_MPP_INCLUDE_DIRS ${_K230_MPP_INCLUDE_DIRS})
set(K230_MPP_LIBRARIES ${_K230_MPP_LIBS})

unset(_K230_MPP_ROOT)
unset(_K230_MPP_INCLUDE_DIRS)
unset(_K230_MPP_LIBS)
