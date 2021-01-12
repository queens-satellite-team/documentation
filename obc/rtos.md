


# **RTOS & Interrupts**

An MCU has a single CPU meaning it must execute functions one at a time. Due to this we need to implement a system that will be able prioritize certain tasks at specific times we need them to run. This will be done through RTos and Interrupts.

## **A Brief Overview**

RTos stands for Real time operating system and basically manages what function the CPU executes due to priority. Since only one function can be active at a single time the RTos selects the function that is trying to run at a time with the highest priority. That means if I have function x and function y both trying to run at the same time, but function y has a higher priority than function x, the MCU will only execute function y.

In addition to this we will need to handle function calls that are event triggered (ex. button pressed) rather than time triggered, to do this we will use the concept of interrupts. There are 2 ways to make an interrupt, the first way is to have the MCU constantly check if the function has been called but this is inefficient. The other, more efficient way is to activate when an electrical signal is received. The MCU does this by using a nested vector interrupt control (NVIC). While that sounds long and complicated, its simple once you figure out what it does. The NVIC takes in an electrical signal from a GPIO output pin on the MCU and then matches the pin number to a table of functions called interrupt service routines (ISR). It would look something like this:  


PIC HERE 1

When the NVIC calls one of theses functions, it basically tells the CPU to stop running all other tasks and run that instead.

The project that will be created using the concepts above will be a circuit consisting of 3 blinking lights. Two lights will blink out of sync with one another and the final one will trigger only once the interrupt has been activated.

## **RTos Implementation**

First open CubeMX and find the MCU you will be working with. If you are unfamiliar with the software, please watch a tutorial before continuing with this documentation or refer to the blinkProgram documentation. From there click on the “Middleware” drop down menu and then FREERTOS. 

PIC HERE 2

Click the Interface drop down menu and select CMSIS_V2

PIC HERE 3

From there click on “Tasks and Queues”. This is the place we will be impleneting our priotiy based functions. You should have a defaultTask currently present.

PIC HERE 4

Click on defaultTask and a menu should pop up. For this project we will only concern ourselves with the name of the task (Task Name), and the priority it has relative to the other tasks (Priority.) Rename the function to “blink1” and hit OK.

PIC HERE 5

Now, the whole point of an RTos is the manage tasks based off their priority so to actually utilize RTos we will need to add more tasks. To do this click Add and the same menu should pop up from before. Change the Task Name to blink2 and the Priority to be osPriorityBelowNormal, which will cause it to have a lower priority realitive to our original blink1 task.

PIC HERE 6

Now before we continue there is a problem with the timer. Both FREERTOS and the HAL functions use the SysTick clock. To fix this, go to “System Core” and click “SYS”. From there click the Timebase Source drop down menu and **change SysTick to TIM1 and check off Debug Serial Wire.**

## **Interrupt Implementation**

For this interrupt the external signal of a button will be used. First, set one of these pins to a GPIO_EXTI, for this example we will be using pin-62 i.e. PB9.

PIC HERE 7

Next, got to the NVIC tab on the left and check the box labeled “EXTI line 4 to 15 interrupts”.

PIC HERE 8

Go to the GPIO tab on the left and then select the pin which was changed to the GPIO_EXTI, which in this case will be PB9, and change the GPIO mode to “External Interrupt Mode with Rising edge trigger detection” which should trigger the interrupt only when the button is initially pressed. In addition to this, select “Pull-down” in the GPIO Pull-up/Pull-down drop down menu to ensure that our circuit will only activate the interrupt when the button is pressed and will see ground otherwise.

PIC HERE 9

## **Circuit**

If you are reading this documentation then I will assume you understand circuit design and will not go over the basic details of the design, but rather just what will be unique to this build. Each LED will be connected to one of the 3 GPIO outputs initialized and then connected to ground. One lead of the button will be connected to ground while the other will be connected to the GPIO_EXTI. The product should look like the image below.

PIC 10 OF CIRCUIT HERE (YOU HAVENT TAKEN THE PHOTO YET)

## CODE - RTOS

Almost all the setup for the RTos part of this project has been done in the CUBEMX initialization. All that needs to be done now is to outline what you would like each thread to do. For this implementation we will have Startblink1() trigger one of the GPIO outputs (which will in turn trigger the corresponding LED) and Startblink2() trigger another one of the GPIO outputs. To make sure the LEDs are flashing at different frequencies a function called osDelay() will be implemented that does the job of delaying the thread it is called in by the number of milliseconds entered into the parameter. Finally, have the startInteruptTask()thread toggle the final LED. Since we will be using startInteruptTask() as the thread that will execute when the interrupt is triggered it will need to be modified in the Interrupt Section.

## CODE - INTERUPT

**The entire method of the implementation of the interrupt involves a thread with the highest possible priority  ( startInteruptTask() )and an interrupt ( HAL_GPIO_EXTI_Callback() ). What happens is that the RTOS runs normally but the startInteruptTask thread is suspended using the function osThreadFlagsWait() which will suspend any thread until a signal has been sent. Within interrupt ( HAL_GPIO_EXTI_Callback() ) there will be an if statement that will execute when the GPIO_Output button from the initial STM32 setup is pressed. What happens when that if statement is executed is the function osThreadFlagsSet() is used and sends the signal that the startInteruptTask thread was waiting for which in turn activates it. After the startInteruptTask is run once it will run back into the osThreadFlagsWait() and be suspended until the interrupt(aka the button) is activated again. The flag will pass as an arguement to both functions will be 0x0002U. The code provided below showcases what is explained above.

**NOTE: Why does it seem like multiple threads are running at the same time?**


I did some tests and they are not. My best guess is that the tasks are suspended and then put into a queue. Once the higher priority thread finishes or encounters an OSdelay() the lower priority function will continue. The fraction of a second interruption of the lower priority function is un able to be noticed by the human eye so it seems as if there are multiple threads running at once. This delay will most likely become more prominent (and more useful) when implantation of functions that require a larger time consumption.
