## Where to find information
The main sources of information should always be the datasheet and the reference manual. The datasheet summarises the physical specs, memory map, etc., whereas the reference manual is about how to use the memory and peripherals. 

Additionally, we can refer to the reference manual for the Arm Cortex M4, and the programming manual.

## `libopencm3`
This is an open-source firmware library for a variety of Arm Cortex-M microcontrollers, intended for use with a GCC Arm toolchain (arm-elf or arm-none-eabi). 

Let’s look at `gpio_mode_setup()`, which takes a GPIO port, a mode, pull up/down mode, and pin(s).q