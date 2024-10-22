## Setup
Just ran these, also got the GCC ARM toolchain and added to path like
`PATH=$PATH:/Applications/ArmGNUToolchain/13.3.rel1/arm-none-eabi/bin`

```bash
# Clone the repo
git clone git@github.com:lowbyteproductions/bare-metal-series.git
cd bare-metal-series

# Initialise the submodules (libopencm3)
git submodule init
git submodule update

# Build libopencm3
cd libopencm3
make
cd ..
```
We’re writing a simple LED blinking program. The first step is to configure the clock. We’ll go for the default for the ARM Cortex F4 seriesat 84 MHz.
```c
#include <libopencm3/stm32/rcc.h>

static void rcc_setup(void) {
rcc_clock_setup_pll(&rcc_hsi_configs[RCC_CLOCK_3V3_84MHZ]);
}
```
Now we can setup our GPIO for our LED. We’ll enable the pin at Port A, pin 5. We’ll use `gpio_mode_setup()` from `stm32/gpio.h` and pass in 
- the port
- mode (output)
- pull up/down (which is like a default value if we don’t set it to high or low)
- the pin(s), with | notation

-> Need to check which port and pin.

Additionally, because all peripherals are disabled by default, we need to enable the clock for port A. 

```c
static void gpio_setup(void) {
  rcc_periph_clock_enable(RCC_GPIOA);
  
  gpio_mode_setup(GPIOA, GPIO_MODE_OUTPUT, GPIO_PUPD_NONE, GPIO5 | GPIO6);
}
```

Now we can call our setup functions and then toggle the LED in the loop. There’s a `gpio_toggle()` function that we’ll use. We also need to build in a small delay so we can see it toggling. 
Additionally, we can just name our ports and pins more clearly using `#define`. 

```c
#define LED_PORT (GPIOA)
#define LED_PIN_1 (GPIO5)
#define LED_PIN_2 (GPIO6)

// now we get
static void gpio_setup(void) {
  rcc_periph_clock_enable(RCC_GPIOA);
  gpio_mode_setup(LED_PORT, GPIO_MODE_OUTPUT, GPIO_PUPD_NONE, LED_PIN_1 | LED_PIN_2);

}
```

Next, let’s write our delay function. We can’t just put in a blank loop so we put in an Assembly no-op instruction.
```c
static void delay_cycles(uint32_t cycles) {

for (uint32_t i = 0; i < cycles; i++) {

  __asm__("nop");

}
}
```

So our main function and loop will look like this. We’ll pause for roughly a second (the clock speed is 84 MHz, each loop does 4 instructions), so 84 million / 4 will be roughly a second.
```c
int main(void) {

  rcc_setup();
  gpio_setup();

  while (1) {
  
    delay_cycles(84000000 / 4);
    gpio_toggle(LED_PORT, LED_PIN_1);
    delay_cycles(84000000 / 4);
    gpio_toggle(LED_PORT, LED_PIN_2);
  
  }

return 0;

}
```
We can now run `cd app && make`. We get an .o (object) file, a .bin file (the code to be loaded onto the chip) and an .elf file. 

Also, we have a .map file, which tells you what ended up in your program after compilation, which can be useful for debugging. 

Also added platformio.ini in the root of `bare-metal-series`:
```
[env:genericSTM32F407VET6]

platform = ststm32
board = genericSTM32F407VET6
framework = libopencm3
debug_tool = stlink

upload_protocol = stlink
upload_port = /dev/cu.wlan-debug
monitor_port = /dev/cu.wlan-debug

[platformio]
src_dir = /Users/maurits/training/courses/youtube/bare-metal-series/app/src
```
