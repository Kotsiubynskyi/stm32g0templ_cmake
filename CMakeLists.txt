cmake_minimum_required(VERSION 3.15)

# Project name and supported languages
project(blink LANGUAGES C CXX ASM)
# Set the C and C++ compilers (arm-none-eabi)
# Specify the target executable
add_executable(${PROJECT_NAME}.elf)

# Add source files
target_sources(${PROJECT_NAME}.elf PRIVATE
    src/main.cpp
    stm32_hal/Drivers/STM32G0xx_HAL_Driver/Src/stm32g0xx_hal.c
    stm32_hal/Drivers/STM32G0xx_HAL_Driver/Src/stm32g0xx_ll_pwr.c
    stm32_hal/Drivers/STM32G0xx_HAL_Driver/Src/stm32g0xx_hal_pwr.c
    stm32_hal/Drivers/STM32G0xx_HAL_Driver/Src/stm32g0xx_hal_pwr_ex.c
    stm32_hal/Drivers/STM32G0xx_HAL_Driver/Src/stm32g0xx_hal_gpio.c
    stm32_hal/Drivers/STM32G0xx_HAL_Driver/Src/stm32g0xx_hal_cortex.c
    stm32_hal/Drivers/CMSIS/Device/ST/STM32G0xx/Source/Templates/system_stm32g0xx.c
    stm32_hal/Drivers/CMSIS/Device/ST/STM32G0xx/Source/Templates/gcc/startup_stm32g030xx.s
)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Set the C and C++ standards
set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 17)

# Include directories for headers
target_include_directories(${PROJECT_NAME}.elf PRIVATE
    include
    stm32_hal/Drivers/STM32G0xx_HAL_Driver/Inc
    stm32_hal/Drivers/CMSIS/Device/ST/STM32G0xx/Include
    stm32_hal/Drivers/CMSIS/Include
)

# Linker script for memory mapping
target_link_options(${PROJECT_NAME}.elf PRIVATE
  #    -T${CMAKE_SOURCE_DIR}/STM32G030F6PX_FLASH.ld
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
    -fno-rtti             # Disable RTTI for bare-metal code
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
