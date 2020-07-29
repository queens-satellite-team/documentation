# Embedded Systems Development
### The Basics
The onboard computer is a type of [embedded system](https://en.wikipedia.org/wiki/Embedded_system).
The main differentiator between a general purpose computer and an embedded system
is that an embedded system does a specific-task rather than run arbitrary
programs. This means embedded systems are usually resource constrained when
compared to a general purpose computer i.e slower clock speed, less memory,
lower power requirements. Developing for an embedded system also requires more
knowledge of the hardware specifics of the chip being used than you may be used
to. Fear not, we will cover the basics.

The most common architecture for embedded systems development is the ARM Cortex
M family of processors. [Arm Holdings](https://en.wikipedia.org/wiki/Arm_Holdings)
designs, develops and licenses their processor design to other companies who
manufacture them. The most common manufacturer of the ARM architecture is
[STMicroelectronics](https://en.wikipedia.org/wiki/STMicroelectronics). In addition
to the ARM architecture specification, ST offers different peripherals for
different boards. Hence there is a large family of processors that are manfuactured
by ST all using a given ARM architecture but with different peripherals. What is
a peripheral? A broad definition is a device that is connected to a computer
but is not part of the core architecture.

#### Startup Code
The startup code provides the reset vector, initial stack pointer value, and a
symbol for each of the interrupt vectors.
- [reset vector](https://en.wikipedia.org/wiki/Reset_vector) is the default
location the CPU goes to find the first instruction it will execute after reset.
- [initial stack pointer value]() is a value at the initial address of the stack.
- [interrupt vector]() are addresses that inform the interrupt handler as to
where to find the interrupt service routine (ISR).

### Development Boards
At the time of writing, I am developing using a [STM32F446 Nucleo-64](https://www.digikey.ca/en/products/detail/stmicroelectronics/NUCLEO-F446RE/5347712?s=N4IgTCBcDaIHIFUDCAZAogeQLQDEAseAbAEpogC6AvkA) development
board. You can read about the benefits of a development board [here](https://en.wikipedia.org/wiki/Microprocessor_development_board).
Specifically, the Nucleo comes with an embedded [ST-Link programmer/debugger](https://www.st.com/en/development-tools/st-link-v2.html)
giving us an Arduino like functionality. The ST-link is what allows us to put
code onto the processor often called "flashing". If you have a more bare-bones
break out board style setup, you will need an external ST-link. See [this](https://www.amazon.ca/STM32-Blue-Pill-Genuine-STM32F103C8T6/dp/B07S2VF1PZ/ref=sr_1_3?dchild=1&keywords=STM32&qid=1595686733&sr=8-3).
for example.


# STM32 and GNU Toolchain Development
### Getting Started
The name of the GNU Arm toolchain is gcc-arm-none-eabi and it can be downloaded
[here](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm/downloads).
For more information about software development toolchains check out this [wiki](https://en.wikipedia.org/wiki/Toolchain).
You will also want to download the reference manual for the development board
you are using, which for me was STM32F445RET6 [here](https://www.st.com/resource/en/reference_manual/dm00135183-stm32f446xx-advanced-arm-based-32-bit-mcus-stmicroelectronics.pdf).
You will need to reference this lengthy manual later when we setup the startup
and linker scripts.

Once you have downloaded and installed the toolchain for your platform, browse
the code and README's in the `share` directory. Particularly under
`share/gcc-arm-none-eabi/samples`, there are 3 directories: `ldscripts`, `src`
and `startup`. From `readme.txt` in `samples` directory:
- `samples/startup` - Startup code needed by samples
- `       /ldscripts` - Shared linker script pieces needed by samples
- `       /src` - Sample source code and Makefile, each sub-dir is a case

Startup contains assembly code needed to
### Common Acronyms
There are lots of acronyms in embedded systems, for an incomplete catalogue
see `acronyms.md`.
### Books
- [The Designer's Guide to the Cortex-M Processor Family: A Tutorial Approach](https://www.amazon.ca/Designers-Guide-Cortex-M-Processor-Family/dp/0080982964/ref=sr_1_1?dchild=1&keywords=The+Designer%27s+Guide+to+the+Cortex-m+Processor+Family.+A+Tutorial+Approach&qid=1595690529&sr=8-1)
### Resources
- [Arm Assembler Language Guide](http://www.keil.com/support/man/docs/armasm/armasm_deb1353593789871.htm)
- [CMSIS Github](https://github.com/ARM-software/CMSIS_5)
- [HAL Documentation](https://www.st.com/resource/en/user_manual/dm00105879-description-of-stm32f4-hal-and-ll-drivers-stmicroelectronics.pdf)
