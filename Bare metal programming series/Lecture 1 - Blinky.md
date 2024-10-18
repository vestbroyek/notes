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
We’re writing a simple LED blinking program. The first step is to configure the clock. 
```c
static void rcc_setup(void) {

  
}

```