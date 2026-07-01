# Copyright (c) 2026 HPMicro
# SPDX-License-Identifier: BSD-3-Clause

cmake_minimum_required(VERSION 3.13)

set(CONFIG_CHERRYUSB 1)
set(CONFIG_USB_DEVICE 1)
set(CONFIG_USB_DEVICE_DFU 1)

if("${HPM_BUILD_TYPE}" STREQUAL "")
  set(HPM_BUILD_TYPE flash_xip)
endif()

# Apply hpm_sdk patch if not already applied
set(HPM_SDK_PATCH_FILE "${CMAKE_CURRENT_SOURCE_DIR}/hpm_sdk_v1.12.0.patch")
if(EXISTS "${HPM_SDK_PATCH_FILE}")
    if(NOT DEFINED ENV{HPM_SDK_BASE})
        set(ENV{HPM_SDK_BASE} "${CMAKE_CURRENT_SOURCE_DIR}/hpm_sdk")
    endif()
    execute_process(
        COMMAND git apply --check --reverse ${HPM_SDK_PATCH_FILE}
        WORKING_DIRECTORY $ENV{HPM_SDK_BASE}
        RESULT_VARIABLE PATCH_ALREADY_APPLIED
        OUTPUT_QUIET ERROR_QUIET
    )
    if(PATCH_ALREADY_APPLIED EQUAL 0)
        message(STATUS "hpm_sdk patch already applied, skipping")
    else()
        message(STATUS "Applying hpm_sdk patch: ${HPM_SDK_PATCH_FILE}")
        execute_process(
            COMMAND git apply --verbose ${HPM_SDK_PATCH_FILE}
            WORKING_DIRECTORY $ENV{HPM_SDK_BASE}
            RESULT_VARIABLE PATCH_RESULT
            OUTPUT_VARIABLE PATCH_OUTPUT
            ERROR_VARIABLE PATCH_ERROR
        )
        if(PATCH_OUTPUT)
            message(STATUS "git apply: ${PATCH_OUTPUT}")
        endif()
        if(PATCH_ERROR)
            message(STATUS "git apply: ${PATCH_ERROR}")
        endif()
        if(NOT PATCH_RESULT EQUAL 0)
            message(FATAL_ERROR "Failed to apply hpm_sdk patch (exit code: ${PATCH_RESULT})")
        else()
            message(STATUS "hpm_sdk patch applied successfully")
        endif()
    endif()
endif()
find_package(hpm-sdk REQUIRED HINTS $ENV{HPM_SDK_BASE})
project(dfu)
sdk_inc(common)
sdk_src(common/hpm_dfu_trigger.c)
sdk_inc(.)
sdk_app_src(src/main.c src/dfu.c)
sdk_compile_definitions(USBD_DFU_APP_DEFAULT_ADD=0x80020000)
sdk_compile_definitions(BOARD_SHOW_CLOCK=0)
sdk_compile_definitions(BOARD_SHOW_BANNER=0)

# Generate IDE projects
generate_ide_projects()
