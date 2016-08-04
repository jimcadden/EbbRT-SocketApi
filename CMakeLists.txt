cmake_minimum_required(VERSION 2.6)
include(CMakePackageConfigHelpers)
set(CMAKE_CXX_FLAGS_DEBUG          "-O0 -g3")
set(CMAKE_CXX_FLAGS_MINSIZEREL     "-Os -DNDEBUG")
set(CMAKE_CXX_FLAGS_RELEASE        "-O4 -flto -DNDEBUG")
set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "-O2 -g3")

project("EbbRT-SocketApi" CXX)

set(INSTALL_INCLUDE_DIR include/)
set(INSTALL_LIB_DIR lib/)
set(INSTALL_CONFIG_DIR ${INSTALL_LIB_DIR}cmake/EbbRT-SocketApi)

set(LIB_SOURCES 
  ${PROJECT_SOURCE_DIR}/SocketApi.cc 
  ${PROJECT_SOURCE_DIR}/SocketManager.cc
  ${PROJECT_SOURCE_DIR}/Vfs.cc
  ${PROJECT_SOURCE_DIR}/lwip/api/netdb.c
  ${PROJECT_SOURCE_DIR}/lwip/core/def.c
  ${PROJECT_SOURCE_DIR}/lwip/core/ip_addr.c
  ${PROJECT_SOURCE_DIR}/lwip/core/memp.c
)
set(LIB_INCLUDES
  ${PROJECT_SOURCE_DIR}
  ${PROJECT_SOURCE_DIR}/gnu/include
  ${PROJECT_SOURCE_DIR}/lwip/etc
  ${PROJECT_SOURCE_DIR}/lwip/include
  ${PROJECT_SOURCE_DIR}/lwip/include/ipv4
  ${PROJECT_SOURCE_DIR}/lwip/include/posix
)

if( ${CMAKE_SYSTEM_NAME} STREQUAL "EbbRT")
  message(STATUS "Building baremetal library...")
  set(CMAKE_CXX_FLAGS 
    "${CMAKE_CXX_FLAGS} -std=gnu++14 -Wall -Werror ") 
  set(CMAKE_CXX_FLAGS 
    "${CMAKE_CXX_FLAGS} -iquote ${PROJECT_SOURCE_DIR}/lwip/include")

  # build & install
  include_directories(${LIB_INCLUDES})
  add_library(EbbRT-SocketApi STATIC ${LIB_SOURCES} )
  install(TARGETS EbbRT-SocketApi 
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib
    )
  install( DIRECTORY ${PROJECT_SOURCE_DIR} DESTINATION
    include/EbbRT-SocketApi FILES_MATCHING PATTERN "*.h")
  install( DIRECTORY ${PROJECT_SOURCE_DIR}/gnu DESTINATION
    include/EbbRT-SocketApi FILES_MATCHING PATTERN "*.h")
  install( DIRECTORY ${PROJECT_SOURCE_DIR}/lwip DESTINATION
    include/EbbRT-SocketApi FILES_MATCHING PATTERN "*.h")

  # cmake pkg config file
  configure_package_config_file(EbbRT-SocketApiConfig.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/EbbRT-SocketApiConfig.cmake
    INSTALL_DESTINATION ${INSTALL_CONFIG_DIR} 
    PATH_VARS INSTALL_INCLUDE_DIR INSTALL_LIB_DIR 
    )
  install( FILES ${CMAKE_CURRENT_BINARY_DIR}/EbbRT-SocketApiConfig.cmake
    DESTINATION ${INSTALL_CONFIG_DIR} 
    )
else()
  message(FATAL_ERROR "System type unsupported: ${CMAKE_SYSTEM_NAME}")
endif()
