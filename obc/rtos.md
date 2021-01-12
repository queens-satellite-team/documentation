


# **RTOS & Interrupts**

An MCU has a single CPU meaning it must execute functions one at a time. Due to this we need to implement a system that will be able prioritize certain tasks at specific times we need them to run. This will be done through RTos and Interrupts.

## **A Brief Overview**

RTos stands for Real time operating system and basically manages what function the CPU executes due to priority. Since only one function can be active at a single time the RTos selects the function that is trying to run at a time with the highest priority. That means if I have function x and function y both trying to run at the same time, but function y has a higher priority than function x, the MCU will only execute function y.

In addition to this we will need to handle function calls that are event triggered (ex. button pressed) rather than time triggered, to do this we will use the concept of interrupts. There are 2 ways to make an interrupt, the first way is to have the MCU constantly check if the function has been called but this is inefficient. The other, more efficient way is to activate when an electrical signal is received. The MCU does this by using a nested vector interrupt control (NVIC). While that sounds long and complicated, its simple once you figure out what it does. The NVIC takes in an electrical signal from a GPIO output pin on the MCU and then matches the pin number to a table of functions called interrupt service routines (ISR). It would look something like this:  


![image1](https://user-images.githubusercontent.com/60119461/104344572-51a82380-54cb-11eb-982b-ff3fe18b6f30.PNG)

When the NVIC calls one of theses functions, it basically tells the CPU to stop running all other tasks and run that instead.

The project that will be created using the concepts above will be a circuit consisting of 3 blinking lights. Two lights will blink out of sync with one another and the final one will trigger only once the interrupt has been activated.

## **RTos Implementation**

First open CubeMX and find the MCU you will be working with. If you are unfamiliar with the software, please watch a tutorial before continuing with this documentation or refer to the blinkProgram documentation. From there click on the “Middleware” drop down menu and then FREERTOS. 

![image2](https://user-images.githubusercontent.com/60119461/104344877-af3c7000-54cb-11eb-9d8c-be0acb1f0950.png)

Click the Interface drop down menu and select CMSIS_V2

![image3](https://user-images.githubusercontent.com/60119461/104344912-ba8f9b80-54cb-11eb-9bf6-3fbc6d0fd418.png)

From there click on “Tasks and Queues”. This is the place we will be impleneting our priotiy based functions. You should have a defaultTask currently present.

![image4](https://user-images.githubusercontent.com/60119461/104344957-c713f400-54cb-11eb-8054-d3f2b7efbc5e.png)

Click on defaultTask and a menu should pop up. For this project we will only concern ourselves with the name of the task (Task Name), and the priority it has relative to the other tasks (Priority.) Rename the function to “blink1” and hit OK.

![image5](https://user-images.githubusercontent.com/60119461/104344973-cbd8a800-54cb-11eb-82b8-361b01bf5beb.png)

Now, the whole point of an RTos is the manage tasks based off their priority so to actually utilize RTos we will need to add more tasks. To do this click Add and the same menu should pop up from before. Change the Task Name to blink2 and the Priority to be osPriorityBelowNormal, which will cause it to have a lower priority realitive to our original blink1 task.

![image6](https://user-images.githubusercontent.com/60119461/104345005-d3984c80-54cb-11eb-8cb6-cef56f06ea0a.png)

Now before we continue there is a problem with the timer. Both FREERTOS and the HAL functions use the SysTick clock. To fix this, go to “System Core” and click “SYS”. From there click the Timebase Source drop down menu and **change SysTick to TIM1 and check off Debug Serial Wire.**

## **Interrupt Implementation**

For this interrupt the external signal of a button will be used. First, set one of these pins to a GPIO_EXTI, for this example we will be using pin-62 i.e. PB9.

![image7](https://user-images.githubusercontent.com/60119461/104345027-dd21b480-54cb-11eb-9938-f757d246e412.png)

Next, got to the NVIC tab on the left and check the box labeled “EXTI line 4 to 15 interrupts”.

![image8](https://user-images.githubusercontent.com/60119461/104345058-e6128600-54cb-11eb-8e04-fd70d3778a8b.png)

Go to the GPIO tab on the left and then select the pin which was changed to the GPIO_EXTI, which in this case will be PB9, and change the GPIO mode to “External Interrupt Mode with Rising edge trigger detection” which should trigger the interrupt only when the button is initially pressed. In addition to this, select “Pull-down” in the GPIO Pull-up/Pull-down drop down menu to ensure that our circuit will only activate the interrupt when the button is pressed and will see ground otherwise.

![image9](https://user-images.githubusercontent.com/60119461/104345091-ef035780-54cb-11eb-9b5c-df338fe7d419.png)

## **Circuit**

If you are reading this documentation then I will assume you understand circuit design and will not go over the basic details of the design, but rather just what will be unique to this build. Each LED will be connected to one of the 3 GPIO outputs initialized and then connected to ground. One lead of the button will be connected to ground while the other will be connected to the GPIO_EXTI. The product should look like the image below.

![image10](https://user-images.githubusercontent.com/60119461/104348096-51aa2280-54cf-11eb-8cdc-be707ec2e244.jpg)

## CODE - RTOS

Almost all the setup for the RTos part of this project has been done in the CUBEMX initialization. All that needs to be done now is to outline what you would like each thread to do. For this implementation we will have Startblink1() trigger one of the GPIO outputs (which will in turn trigger the corresponding LED) and Startblink2() trigger another one of the GPIO outputs. To make sure the LEDs are flashing at different frequencies a function called osDelay() will be implemented that does the job of delaying the thread it is called in by the number of milliseconds entered into the parameter. Finally, have the startInteruptTask()thread toggle the final LED. Since we will be using startInteruptTask() as the thread that will execute when the interrupt is triggered it will need to be modified in the Interrupt Section.

    void Startblink1(void *argument)
    {
      /* USER CODE BEGIN 5 */
      /* Infinite loop */
      for(;;)
      {
    	  HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_6);
        osDelay(600);
      }
      /* USER CODE END 5 */
    }

    void Startblink2(void *argument)
    {
      /* USER CODE BEGIN Startblink2 */
      /* Infinite loop */
      for(;;)
      {
    	  HAL_GPIO_TogglePin(GPIOB, GPIO_PIN_1);
        osDelay(500);
      }
      /* USER CODE END Startblink2 */
    }

## CODE - INTERUPT

**The entire method of the implementation of the interrupt involves a thread with the highest possible priority  ( startInteruptTask() )and an interrupt ( HAL_GPIO_EXTI_Callback() ). What happens is that the RTOS runs normally but the startInteruptTask thread is suspended using the function osThreadFlagsWait() which will suspend any thread until a signal has been sent. Within interrupt ( HAL_GPIO_EXTI_Callback() ) there will be an if statement that will execute when the GPIO_Output button from the initial STM32 setup is pressed. What happens when that if statement is executed is the function osThreadFlagsSet() is used and sends the signal that the startInteruptTask thread was waiting for which in turn activates it. After the startInteruptTask is run once it will run back into the osThreadFlagsWait() and be suspended until the interrupt(aka the button) is activated again. The flag will pass as an arguement to both functions will be 0x0002U. The code provided below showcases what is explained above.

    /* USER CODE BEGIN 4 */
    	void HAL_GPIO_EXTI_Callback( uint16_t GPIO_Pin)
    	{
    		if ( GPIO_Pin == GPIO_PIN_9)
    		{
    			osThreadFlagsSet(interruptTaskHandle,0x00000001U);
    		}
    		else __NOP();
    	}
    
    
    /* USER CODE END 4 */
    
    
    void StartInterruptTask(void *argument)
    {
      /* USER CODE BEGIN StartInterruptTask */
      /* Infinite loop */
    
    	for(;;)
    	{
    		osThreadFlagsWait(0x00000001U,osFlagsWaitAny,osWaitForever);
    		HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_3);
    
    	}
      /* USER CODE END StartInterruptTask */
    }

**NOTE: Why does it seem like multiple threads are running at the same time?**


I did some tests and they are not. My best guess is that the tasks are suspended and then put into a queue. Once the higher priority thread finishes or encounters an osDelay() the lower priority function will continue. The fraction of a second interruption of the lower priority function is un able to be noticed by the human eye so it seems as if there are multiple threads running at once. This delay will most likely become more prominent (and more useful) when implantation of functions that require a larger time consumption.
