cmake_minimum_required (VERSION 2.8)

project (rffe-fw)

set(CMAKE_SHARED_LIBRARY_LINK_C_FLAGS "")

include(toolchain/arm_none_eabi_toolchain.cmake)

set(TOOLCHAIN "GCC_ARM" CACHE STRING "Toolchain used to build project")

set(MBED_PATH mbed-lib)
set(MBED_MOUNT /media/MBED/ CACHE PATH "Path to MBED Storage")

set(MBED_TARGET LPC1768 CACHE STRING "MBED Target CPU")

set(USE_RTOS true CACHE BOOL "Compile MBED-RTOS lib")
set(USE_NET true CACHE BOOL "Compile EthernetInterface lib")
set(USE_USBHOST false CACHE BOOL "Compile USBHost lib")

if(USE_NET)
  set(USE_DHCP true CACHE BOOL "Use DHCP for Eth interface")
  set(USE_FIXIP false CACHE BOOL "Use Fixed IP for Eth interface")
  if(USE_DHCP AND USE_FIXIP)
    message(FATAL_ERROR "USE_DHCP and USE_FIXIP were selected. Choose one!")
  endif()
  if(USE_DHCP)
    message(STATUS "DHCP Interface selected!")
    add_definitions("-DETH_DHCP")
  endif()
  if(USE_FIXIP)
    set(MBED_IP "10.0.18.59" CACHE STRING "MBED IP Addr")
    set(MBED_MASK "255.255.255.0" CACHE STRING "MBED IP Mask")
    set(MBED_GATEWAY "10.0.18.1" CACHE STRING "MBED Gateway IP")
    message(STATUS "Fixed IP Interface selected!\n IP:${MBED_IP} \n Mask:${MBED_MASK} \n Gateway:${MBED_GATEWAY}\n")
    add_definitions("-DETH_FIXIP")
    add_definitions("-DETH_IP=\"${MBED_IP}\"")
    add_definitions("-DETH_MASK=\"${MBED_MASK}\"")
    add_definitions("-DETH_GATEWAY=\"${MBED_GATEWAY}\"")
  endif()
endif()

include(mbed.cmake)

#Add all project sources to be compiled
set(PROJ_SRCS main.cpp)
include_directories(${CMAKE_CURRENT_SOURCE_DIR})

include_directories(${PROJ_HDRS})
add_executable(${CMAKE_PROJECT_NAME} ${PROJ_SRCS} ${MBED_OBJECTS})

target_link_libraries(${CMAKE_PROJECT_NAME} ${MBED_LIBS} )

set_target_properties(${CMAKE_PROJECT_NAME} PROPERTIES
  SUFFIX ".axf"
  )

#Create binary file from axf
add_custom_command(TARGET ${CMAKE_PROJECT_NAME} POST_BUILD
  COMMAND ${CMAKE_OBJCOPY} -O binary ${CMAKE_PROJECT_NAME}.axf ${CMAKE_PROJECT_NAME}.bin
  WORKING_DIRECTORY ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}
  COMMENT "Converting the AXF output to a binary file"
  )

# ------------------------------------------------------------------------------
# custom target for copying to mbed device
add_custom_target(upload
  COMMAND cp ${CMAKE_PROJECT_NAME}.bin ${MBED_MOUNT}
)

# ------------------------------------------------------------------------------
# custom target for opening serial console
add_custom_target(sercon
  command screen ${SERCON} 9600
)

