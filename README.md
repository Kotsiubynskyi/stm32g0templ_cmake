Instructions written for Ubuntu (Debian)

## How to build and flash

0. Install needed ARM development toolchain and other utilities:
    ```bash
    sudo apt update && sudo apt install -y cmake git ninja-build gcc-arm-none-eabi stlink-tools
    ```
1. Important: current repository uses submodules so please clone the project ***recursively***:
    ```bash
    git clone --recursive --depth 1 https://github.com/Kotsiubynskyi/stm32g0templ_cmake.git blink
    ```
2. Generate build files:
    ```bash
    cd blink
    mkdir build
    cd build
    cmake .. -DCMAKE_TOOLCHAIN_FILE=STM32G0_toolchain.cmake -G Ninja
    ```
3. build from ***build*** folder:
    ```bash
    ninja
    ```
4. Insert ST-LINK programmer USB stick and make sure it's visible by OS:
    ```bash
    lsusb|grep ST-LINK
    ```

5. upload firmware to MCU:
    ```bash
    st-flash write blink.bin 0x8000000
    ```
