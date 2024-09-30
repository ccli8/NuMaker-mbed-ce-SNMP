# Copyright (c) 2022 ARM Limited. All rights reserved.
# SPDX-License-Identifier: Apache-2.0

add_library(lwip-snmp INTERFACE)

target_include_directories(lwip-snmp
    INTERFACE
        .
        lwip/src/apps/snmp
        lwip/src/include/lwip/apps
)

# NOTE: Ignore below
# lwip/src/apps/snmp/snmp_netconn.c
# lwip/src/apps/snmp/snmp_raw.c
target_sources(lwip-snmp
    INTERFACE
        lwip/src/apps/snmp/snmp_asn1.c
        lwip/src/apps/snmp/snmp_core.c
        lwip/src/apps/snmp/snmp_mib2.c
        lwip/src/apps/snmp/snmp_mib2_icmp.c
        lwip/src/apps/snmp/snmp_mib2_interfaces.c
        lwip/src/apps/snmp/snmp_mib2_ip.c
        lwip/src/apps/snmp/snmp_mib2_snmp.c
        lwip/src/apps/snmp/snmp_mib2_system.c
        lwip/src/apps/snmp/snmp_mib2_tcp.c
        lwip/src/apps/snmp/snmp_mib2_udp.c
        lwip/src/apps/snmp/snmp_msg.c
        lwip/src/apps/snmp/snmp_pbuf_stream.c
        lwip/src/apps/snmp/snmp_scalar.c
        lwip/src/apps/snmp/snmp_snmpv2_framework.c
        lwip/src/apps/snmp/snmp_snmpv2_usm.c
        lwip/src/apps/snmp/snmp_table.c
        lwip/src/apps/snmp/snmp_threadsync.c
        lwip/src/apps/snmp/snmp_traps.c
        lwip/src/apps/snmp/snmpv3.c
        lwip/src/apps/snmp/snmpv3_mbedtls.c
)

target_link_libraries(lwip-snmp
    INTERFACE
        mbed-lwipstack
)

# Don't use IN_LIST with MBED_CONFIG_DEFINITIONS in generator expression
# form, or get the error:
#
# $<IN_LIST> generator expression requires 2 comma separated parameters,
# but got more instead.
#
# This is because there are comma in expanded MBED_CONFIG_DEFINITIONS,
# which break the cmake syntax.
if("LWIP_SNMP_V3=1" IN_LIST MBED_CONFIG_DEFINITIONS)
    target_sources(lwip-snmp
        INTERFACE
            lwip/src/apps/snmp/snmpv3_dummy.c
    )

    target_link_libraries(lwip-snmp
        INTERFACE
            mbed-mbedtls
    )

    target_compile_definitions(mbed-mbedtls
        PUBLIC
            "MBEDTLS_SHA1_C"
            "MBEDTLS_MD5_C"
            "MBEDTLS_DES_C"
            "MBEDTLS_CIPHER_MODE_CFB"
    )
endif()