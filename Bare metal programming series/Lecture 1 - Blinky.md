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

