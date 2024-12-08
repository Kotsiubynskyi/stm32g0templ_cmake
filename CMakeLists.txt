cmake_minimum_required(VERSION 3.15)

set(CMAKE_TOOLCHAIN_FILE "STM32G0_toolchain.cmake")
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Project name and supported languages
project(blink LANGUAGES C CXX ASM)
# Set the C and C++ compilers (arm-none-eabi)
# Specify the target executable
add_executable(${PROJECT_NAME}.elf)

# Add source files
target_sources(${PROJECT_NAME}.elf PRIVATE
    src/main.cpp
    stm32_g0_hal/Drivers/STM32G0xx_HAL_Driver/Src/stm32g0xx_hal.c
    stm32_g0_hal/Drivers/STM32G0xx_HAL_Driver/Src/stm32g0xx_hal_pwr.c
    stm32_g0_hal/Drivers/STM32G0xx_HAL_Driver/Src/stm32g0xx_hal_pwr_ex.c
    stm32_g0_hal/Drivers/STM32G0xx_HAL_Driver/Src/stm32g0xx_hal_gpio.c
    stm32_g0_hal/Drivers/STM32G0xx_HAL_Driver/Src/stm32g0xx_hal_cortex.c
    stm32_g0_hal/Drivers/CMSIS/Device/ST/STM32G0xx/Source/Templates/system_stm32g0xx.c
    stm32_g0_hal/Drivers/CMSIS/Device/ST/STM32G0xx/Source/Templates/gcc/startup_stm32g030xx.s
)

# Include directories for headers
target_include_directories(${PROJECT_NAME}.elf PRIVATE
    include
    stm32_g0_hal/Drivers/STM32G0xx_HAL_Driver/Inc
    stm32_g0_hal/Drivers/CMSIS/Device/ST/STM32G0xx/Include
    stm32_g0_hal/Drivers/CMSIS/Include
)

# Linker script for memory mapping
target_link_options(${PROJECT_NAME}.elf PRIVATE
    -Wl,--gc-sections  # Remove unused sections
    -Wl,-Map=${PROJECT_NAME}.map # Generate a map file
)

# Define compile options
target_compile_options(${PROJECT_NAME}.elf PRIVATE
    -mcpu=cortex-m0
    -mthumb
    -Os                   # Optimize for size
    -ffunction-sections   # Place functions in separate sections
    -fdata-sections       # Place data in separate sections
    -fno-exceptions       # Disable exceptions for bare-metal code
)

# Define preprocessor macros
target_compile_definitions(${PROJECT_NAME}.elf PRIVATE
  -DSTM32G030xx  # Define the specific MCU
)

# Linker flags for Cortex-M0 architecture
target_link_options(${PROJECT_NAME}.elf PRIVATE
    -mcpu=cortex-m0
    -mthumb
    -static
    -Wl,--start-group -lc -lm -Wl,--end-group
)

# Convert .elf firmware to .bin after build
add_custom_command(TARGET ${PROJECT_NAME}.elf POST_BUILD
    COMMAND ${CMAKE_OBJCOPY} -O binary ${PROJECT_NAME}.elf ${PROJECT_NAME}.bin
    COMMENT "Generating ${PROJECT_NAME}.bin from ${PROJECT_NAME}.elf"
)

# Print firmware Flash and RAM usage
add_custom_command(TARGET ${PROJECT_NAME}.elf POST_BUILD
    COMMAND ${CMAKE_SIZE} ${PROJECT_NAME}.elf
    COMMENT "Displaying firmware size"
)
