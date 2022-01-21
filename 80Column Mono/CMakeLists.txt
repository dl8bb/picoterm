cmake_minimum_required(VERSION 3.13)

#set(PICO_SDK_PATH /home/pi//pico/pico/pico-sdk)
# Pull in SDK (must be before project)
include(pico_sdk_import.cmake)

project(picoterm C CXX ASM)
set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 17)

if (PICO_SDK_VERSION_STRING VERSION_LESS "1.3.0")
    message(FATAL_ERROR "Raspberry Pi Pico SDK version 1.3.0 (or later) required. Your version is ${PICO_SDK_VERSION_STRING}")
endif()

set(PICO_TERMINAL_PATH ${PROJECT_SOURCE_DIR})

include(pico_extras_import.cmake)

pico_sdk_init()

#project(picoterm VERSION 1.0.0)
add_compile_options(-Wall
    -Wno-format          # int != int32_t as far as the compiler is concerned because gcc has int32_t as long int
    -Wno-unused-function # we have some for the docs that aren't called
    -Wno-maybe-uninitialized
    -Wno-psabi
    -DLOCALISE_DE
)
add_executable(picoterm
    main.c
    picoterm.c
    speccyfont.h
    tusb_config.h
    keycode_to_ascii.h
    font8.c
)

pico_enable_stdio_usb(picoterm 0)
pico_enable_stdio_uart(picoterm 0)

target_include_directories(picoterm PRIVATE ${CMAKE_CURRENT_LIST_DIR})
# I think the above is necessary to include tusb_config.h

pico_add_extra_outputs(picoterm)

target_compile_definitions(picoterm PRIVATE
    PICO_SCANVIDEO_SCANLINE_BUFFER_COUNT=4
    PICO_SCANVIDEO_PLANE1_FIXED_FRAGMENT_DMA=true 
)

target_link_libraries(picoterm PRIVATE
    pico_scanvideo_dpi
    pico_multicore
    pico_stdlib

    #hardware_gpio
    #hardware_i2c
    #hardware_adc
    #hardware_uart
    #hardware_irq

    #tinyusb_device
    tinyusb_board
    tinyusb_host
)
