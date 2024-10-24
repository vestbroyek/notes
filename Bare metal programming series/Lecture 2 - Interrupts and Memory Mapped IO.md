## Where to find information
The main sources of information should always be the datasheet and the reference manual. The datasheet summarises the physical specs, memory map, etc., whereas the reference manual is about how to use the memory and peripherals. 

Additionally, we can refer to the reference manual for the Arm Cortex M4, and the programming manual.

## `libopencm3`
This is an open-source firmware library for a variety of Arm Cortex-M microcontrollers, intended for use with a GCC Arm toolchain (arm-elf or arm-none-eabi). 

Let’s look at `gpio_mode_setup()`, which takes a GPIO port, a mode, pull up/down mode, and pin(s).

We can see it does a lot of work under the hood using preprocessor directives, which ultimately define constants that correspond to memory locations:
```c
void gpio_mode_setup(uint32_t gpioport, uint8_t mode, uint8_t pull_up_down, uint16_t gpios)
{
  
  uint16_t i;
  
  uint32_t moder, pupd;
  
  
  moder = GPIO_MODER(gpioport);

  // ... 

}

// leads to
#define GPIO_MODER(port) MMIO32((port) + 0x00)
#define GPIOA_MODER GPIO_MODER(GPIOA)

// leads to
#define GPIOA GPIO_PORT_A_BASE
  
// leads to
#define GPIO_PORT_A_BASE (PERIPH_BASE_AHB1 + 0x0000)

// leads to
#define PERIPH_BASE_AHB1 (PERIPH_BASE + 0x20000)
#define PERIPH_BASE (0x40000000U)
```

Notice that MMIO32 just casts whatever address you pass it to a 32-bit integer. 

`libopencm3` does all of this work for us (not just when it comes to GPIO).

## Improving our script and interrupts
Instead of doing useless loops, we can keep track of time using the “sys tick”. 

To do so, we’ll set the frequency. We know the clock runs at 84 MHz, so 84 million cycles per second. For us, milliseconds is fine-grained enough. We pass in a sys tick frequency and the existing CPU frequency. Then we need to enable the counter. Finally, we need to work out how the sys tick will let us know what it is doing. It will do this using an **interrupt**. 

> An interrupt is a mechanism that will temporarily halt the main program’s execution to service an event (like a button press or sensor signal).



