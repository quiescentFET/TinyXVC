#
# Download and export official FTDI's prebuilt d2xx driver as IMPORTED library
#

if (CMAKE_SYSTEM_NAME STREQUAL "Linux")
    if (TXVC_ARCH STREQUAL "amd64")
        set(D2XX_URL "https://ftdichip.com/wp-content/uploads/2022/07/libftd2xx-x86_64-1.4.27.tgz")
    elseif(TXVC_ARCH STREQUAL "arm64")
        set(D2XX_URL "https://ftdichip.com/wp-content/uploads/2025/03/libftd2xx-linux-arm-v8-1.4.33.tgz")
    endif()
    
    set(D2XX_DIR ${CMAKE_CURRENT_BINARY_DIR})
    set(D2XX_ARCHIVE ${D2XX_DIR}/libftd2xx.tar.gz)

    if (NOT EXISTS ${D2XX_ARCHIVE})
        message(STATUS "Downloading libd2xx")
        file(DOWNLOAD ${D2XX_URL} ${D2XX_ARCHIVE}
            SHOW_PROGRESS
            STATUS D2XX_DOWNLOAD_STATUS
            TLS_VERIFY ON
            TIMEOUT 60)
        list(GET D2XX_DOWNLOAD_STATUS 0 D2XX_DOWNLOAD_STATUS_CODE)
        list(GET D2XX_DOWNLOAD_STATUS 1 D2XX_DOWNLOAD_STATUS_ERROR)
        if (NOT (D2XX_DOWNLOAD_STATUS_CODE EQUAL 0))
            message(FATAL_ERROR "Failed to download libd2xx: ${D2XX_DOWNLOAD_STATUS_ERROR}")
        endif()
        message(STATUS "Unpacking libd2xx")
        execute_process(COMMAND tar zx
            WORKING_DIRECTORY ${D2XX_DIR}
            INPUT_FILE ${D2XX_ARCHIVE}
            RESULT_VARIABLE D2XX_UNPACK_STATUS_CODE
            TIMEOUT 60)
        if (NOT (D2XX_UNPACK_STATUS_CODE EQUAL 0))
            message(FATAL_ERROR "Failed to unpack libd2xx")
        endif()
    endif()

    add_library(libftd2xx STATIC IMPORTED GLOBAL)
    set_target_properties(libftd2xx
        PROPERTIES
        IMPORTED_LOCATION ${D2XX_DIR}/linux-arm-v8/libftd2xx-static.a
        INTERFACE_INCLUDE_DIRECTORIES ${D2XX_DIR}/linux-arm-v8
        INTERFACE_COMPILE_DEFINITIONS FTD2XX_STATIC
        INTERFACE_LINK_LIBRARIES pthread)
    return()
    
endif()

message(FATAL_ERROR "Do not know where to find D2XX driver ")

