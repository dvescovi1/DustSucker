cmake_minimum_required(VERSION 3.22)
set(CMAKE_TOOLCHAIN_FILE ${CMAKE_CURRENT_SOURCE_DIR}/stm32-cmake/cmake/stm32_gcc.cmake)
#set(CMAKE_TOOLCHAIN_FILE ${CMAKE_CURRENT_SOURCE_DIR}/cmake/gcc-arm-none-eabi.cmake)
#include("cmake/gcc-arm-none-eabi.cmake")

set(CMAKE_EXPORT_COMPILE_COMMANDS TRUE)


project(DustSucker ASM C CXX)
set(STM32_CUBE_L4_PATH ${CMAKE_CURRENT_SOURCE_DIR}/STM32CubeL4)
#Use freertos from STM32CubeL4
set(FREERTOS_PATH ${STM32_CUBE_L4_PATH}/Middlewares/Third_Party/FreeRTOS)

#Find CMSIS
find_package(HAL COMPONENTS STM32L4 REQUIRED)
find_package(CMSIS COMPONENTS STM32L4 REQUIRED )


find_package(FreeRTOS COMPONENTS ARM_CM33)


add_library(baseHAL INTERFACE)
target_link_libraries(baseHAL INTERFACE
    HAL::STM32::L4
    CMSIS::STM32::L4
    STM32::NoSys
    )
add_library(freertos INTERFACE)
target_link_libraries(freertos INTERFACE
    FreeRTOS::Timers
    FreeRTOS::Heap::1
    FreeRTOS::ARM_CM33
    )

add_executable(dustSucker Src/main.cpp)
target_include_directories(dustSucker PRIVATE Inc)
target_link_libraries(dustSucker PRIVATE
    baseHAL
    freertos
    HAL::STM32::L4::M33::GPIO
    HAL::STM32::L4::M33::RCC
    )
ADD_CUSTOM_TARGET(do_always ALL COMMAND ${CMAKE_OBJCOPY} -O ihex dustSucker.elf dustSucker.hex
                                DEPENDS dustSucker)
