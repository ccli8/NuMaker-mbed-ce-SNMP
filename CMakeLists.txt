# Copyright (c) 2022 ARM Limited. All rights reserved.
# SPDX-License-Identifier: Apache-2.0

cmake_minimum_required(VERSION 3.19)
cmake_policy(VERSION 3.19)

set(MBED_OS_PATH ${CMAKE_CURRENT_SOURCE_DIR}/mbed-os CACHE INTERNAL "")

# Set path for mbed_app.json5
set(MBED_APP_JSON_PATH mbed_app.json5)

# Set path for custom_targets.json5
set(CUSTOM_TARGETS_PATH custom_targets) 
set(CUSTOM_TARGETS_JSON_PATH ${CUSTOM_TARGETS_PATH}/custom_targets.json5) 

# Load Mbed CE toolchain file and basic build system
include(${MBED_OS_PATH}/tools/cmake/app.cmake)

set(APP_PROJECT NUMAKER_MBED_CE_SNMP)
set(APP_TARGET NuMaker-mbed-ce-SNMP)

# Set up project name
project(${APP_PROJECT})

# Add for custom targets
add_subdirectory(${CUSTOM_TARGETS_PATH})

# Add Mbed OS library
add_subdirectory(${MBED_OS_PATH})

# Add other libraries
add_subdirectory(lwip-snmp)

add_executable(${APP_TARGET})

target_include_directories(${APP_TARGET}
    PRIVATE
        app-snmp
        app-snmp/config
        app-snmp/mib
        app-snmp/transport
)

target_sources(${APP_TARGET}
    PRIVATE
        app-snmp/main.cpp
        app-snmp/mib/snmp_gpio_perif_mib.cpp
        app-snmp/transport/snmp_netconn_mbed.cpp
        pre-main/host-stdin/dispatch_host_command.cpp
        pre-main/host-stdin/fetch_host_command.cpp
        pre-main/host-stdin/mem_stats.cpp
        pre-main/host-stdin/pump_host_command.cpp
        pre-main/mbed_main.cpp
)

if("NUVOTON" IN_LIST MBED_TARGET_LABELS)
    target_sources(${APP_TARGET}
        PRIVATE
            "targets/TARGET_NUVOTON/platform_entropy.cpp"
    )
endif()

target_link_libraries(${APP_TARGET}
    PRIVATE
        mbed-os        
        mbed-netsocket-api
        $<$<IN_LIST:MBED_CONF_TARGET_NETWORK_DEFAULT_INTERFACE_TYPE=WIFI,${MBED_TARGET_DEFINITIONS}>:mbed-wifi>
        lwip-snmp
        mbed-lwipstack
        mbed-mbedtls
)

# Must call this for each target to set up bin file creation, code upload, etc
mbed_set_post_build(${APP_TARGET})

# Make sure this is the last line of the top-level build script
mbed_finalize_build()
