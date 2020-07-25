Notes taken form [The Designer's Guide to the Cortex-M Processor Family: A Tutorial Approach](https://www.amazon.ca/Designers-Guide-Cortex-M-Processor-Family/dp/0080982964)
# ARM Cortex M Family features
- CoreSight debug architecture. Allows eight hardware break points. M3 and M4 have
data watchpoint and trace (DWT) unit and instrumentation trace macrocell (ITM).
- DWT can view memory and peripheral registers on the fly without halting CPU.
- ITM provides debug communication between running code and debugger user interface.
# ARM Cortex-M3
- Cortex-M3 is most common processor in Cortex-M processor families. Typically
good balance between performance, power and cost.
- 32 bit CPU, RISC instruction set.
- Separate fetch, decode, and execute units. Meaning, while one instruction is
being executed, another is being decoded and another is being fetched. When
branching the pipeline must be flushed, meaning there is a branch penalty.
- Nested Vector Interrupt Unit (NVIC) services up to 240 interrupt sources. Also
contains 24 bit countdown timer with an auto reload called the systick timer.
Used to provide regular periodic interrupts.
- Wake up Interrupt Controller (WIC). Used to wake up processor from low-power
mode.
# ARM Cortex-M4
