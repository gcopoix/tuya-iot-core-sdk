cmake_minimum_required( VERSION 3.5.0 )
project( tuya-iot-core-sdk LANGUAGES C )

# include Tuya IoT Core SDK (https://developer.tuya.com/en/docs/iot/link-sdk?id=Kaiuyfihfgkr5)
add_subdirectory( tuya-iot-core-sdk )

# Tuya SDK modifications #1: Remove SDK examples from "all" build
set_target_properties(custom_protocol_basic_demo PROPERTIES EXCLUDE_FROM_ALL ON)
set_target_properties(data_model_basic_demo      PROPERTIES EXCLUDE_FROM_ALL ON)
set_target_properties(ota_basic_demo             PROPERTIES EXCLUDE_FROM_ALL ON)
set_target_properties(subdevice_basic_demo       PROPERTIES EXCLUDE_FROM_ALL ON)

# Tuya SDK modifications #2: Remove SDK posix platform_port sources for cross builds
if (${CMAKE_CROSSCOMPILING})
    get_target_property(PLATFORM_SOURCES platform_port SOURCES)
    list(REMOVE_ITEM PLATFORM_SOURCES mbedtls_sockets_wrapper.c # wraps mbedtls->sockets - platform specific
                                      #network_wrapper.c        # wraps tls->mbedtls, can be kept
                                      storage_wrapper.c         # wraps data save/restore - platform specific
                                      system_wrapper.c          # wraps various system calls - platform specific
    )
    set_property(TARGET platform_port PROPERTY SOURCES ${PLATFORM_SOURCES})
endif()

# Tuya SDK modifications #3: Fix public headers of utils_modules (required by install())
set(UTILS_HEADER      "tuya-iot-core-sdk/utils/cJSON.h"
                      "tuya-iot-core-sdk/utils/log.h"
                      "tuya-iot-core-sdk/utils/tuya_error_code.h"
                      "tuya-iot-core-sdk/utils/tuya_log.h"
)
set_property(TARGET utils_modules PROPERTY PUBLIC_HEADER ${UTILS_HEADER})

# Tuya SDK modifications #4: Fix public headers of platform_port (required by install())
#file( GLOB_RECURSE MBEDTLS_HEADER
#    RELATIVE ${CMAKE_CURRENT_SOURCE_DIR}/tuya-iot-core-sdk/libraries/mbedtls/include
#    ${CMAKE_CURRENT_SOURCE_DIR}/tuya-iot-core-sdk/libraries/mbedtls/include/*.h
#)
set(PLATFORM_HEADER   "tuya-iot-core-sdk/interface/system_interface.h"
                      "tuya-iot-core-sdk/interface/storage_interface.h"
                      "tuya-iot-core-sdk/interface/network_interface.h"
                      #${MBEDTLS_HEADER}
)
set_property(TARGET platform_port PROPERTY PUBLIC_HEADER ${PLATFORM_HEADER})

# Tuya SDK modifications #5: Fix public headers of link_core (required by install())
set(CORE_HEADER       "tuya-iot-core-sdk/examples/data_model_basic_demo/tuya_cacert.h"
                      "tuya-iot-core-sdk/include/tuyalink_core.h"
                      "tuya-iot-core-sdk/include/tuya_endpoint.h"
)
set_property(TARGET link_core PROPERTY PUBLIC_HEADER ${CORE_HEADER})

# Tuya SDK modifications #6: Fix public headers of middleware_implementation (required by install())
set(MIDDLEWARE_HEADER #"tuya-iot-core-sdk/interface/http_client_interface.h"
                      "tuya-iot-core-sdk/interface/mqtt_client_interface.h"
)
set_property(TARGET middleware_implementation PROPERTY PUBLIC_HEADER ${MIDDLEWARE_HEADER})

# finally, create 1 tuya-iot-core-sdk library from libraries above
add_library(${PROJECT_NAME} STATIC $<TARGET_OBJECTS:link_core>
                                   $<TARGET_OBJECTS:utils_modules>
                                   $<TARGET_OBJECTS:platform_port>
                                   $<TARGET_OBJECTS:middleware_implementation>)

# tuya-iot-core-sdk library include directories from libraries above
target_include_directories( ${PROJECT_NAME}
    PUBLIC $<TARGET_PROPERTY:link_core,INCLUDE_DIRECTORIES>
           $<TARGET_PROPERTY:middleware_implementation,INCLUDE_DIRECTORIES>
           $<TARGET_PROPERTY:platform_port,INCLUDE_DIRECTORIES>
           $<TARGET_PROPERTY:utils_modules,INCLUDE_DIRECTORIES>
           tuya-iot-core-sdk/examples/data_model_basic_demo
)

# tuya-iot-core-sdk library public headers from libraries above
set_property(TARGET ${PROJECT_NAME}
    PROPERTY PUBLIC_HEADER ${CORE_HEADER}
                           ${UTILS_HEADER}
                           ${PLATFORM_HEADER}
                           ${MIDDLEWARE_HEADER}
)

# tuya-iot-core-sdk library install targets
install(TARGETS ${PROJECT_NAME}
    EXPORT ${PROJECT_NAME}
)
install(TARGETS ${PROJECT_NAME}
    EXPORT ${PROJECT_NAME}
)
# install flattens public headers, so use this method to preserver mbedtls headers in subfolder
install(DIRECTORY tuya-iot-core-sdk/libraries/mbedtls/include/mbedtls
    DESTINATION include
    COMPONENT Headers
)
#install(TARGETS link_core utils_modules platform_port middleware_implementation)
