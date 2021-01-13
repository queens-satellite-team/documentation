# **I2C**

### What is it?

![image1](https://user-images.githubusercontent.com/60119461/97533841-5ad72900-198f-11eb-9608-911bfeee7988.png)

I2C communication involves a connection where (usually 1) master is connected to 1 or more slaves.  The Serial Data (SDA) is the line for the master and slave to share information between one another. The SCL (Serial Clock) is the line which sends the clock signal which essentially tells each of the devices when they should be sending and receiving data from the SDA line.

![image1.5](https://user-images.githubusercontent.com/60119461/97534184-f23c7c00-198f-11eb-9402-a572ac511b48.png)

Expanding on the first diagram, this is a more complex view of how I2C works on a circuit schematic level. One key thing to note about this picture is that there are multiple slaves connected to a single master. When analyzing this, we can understand that each slave and master are all connected to the same SDA line SCL line. The VDD and Rp resistors are used as a pull-up resistor, which basically pulls the circuit up to the voltage VDD, the reasons for this will be discussed below.
 
![image2](https://user-images.githubusercontent.com/60119461/97534233-041e1f00-1990-11eb-94bc-25f2f94cc4f4.png)

Zooming in even further we get this schematic. This one depicts the relationship between a single slave and master on the SDA line. The pull up resistor prevents what is called a hanging node since by default the switches are closed. If there is no voltage and no ground, the circuit will have a hanging node meaning the device will get strange readings. To fix this we place a pull up resistor which brings the connected circuit up to a specific voltage when the circuit is open. This is because there will be no current passing through the resistor when it is open because the circuit is not complete making both node before and after the resistor the same voltage. Another way of looking at this is that since there is no voltage drop across the resistor and we have an applied voltage to the circuit, both nodes must be the same voltage.

![image3](https://user-images.githubusercontent.com/60119461/97534267-113b0e00-1990-11eb-8f4b-052619762f42.png)
![image4](https://user-images.githubusercontent.com/60119461/97534286-1b5d0c80-1990-11eb-8cce-8e13d5a5a4d0.png)


The circuit is essentially able to send information through opening and closing the switch that is just above ground. Slave and Master don’t directly change the signal/ Voltage on the line, they open and close a switch by sending a voltage to it (the switch is a simplification, it is usually replaced by a transistor like a MOSFET which will act as a closed switch when voltage is applied and an open one when voltage is no longer applied.) This allows them to send a series of high and low signals which can be then interpreted as information.

### How Data is Transmitted

![image5](https://user-images.githubusercontent.com/60119461/97534363-3b8ccb80-1990-11eb-8010-614a937511da.png)

This is what a series of those Highs and Lows would look like on the SDA line as this represents the voltage across the SDA line and the SCL line and essentially shows how data is transferred between a slave and a master. Breaking down the diagram from left to right, first the start condition is that Serial Data line (SDL) is pulled down then the Serial Clock Line (SCL) is pulled down (which means they have a LOW value or they are both grounded).

Then, on the SDA line Data is sent from master to data line in the form of 8 bits. The first 7 bits are the address of the slave the master is looking for(in the case of the picture above it will be 1100100) and the last bit is the “read/write” bit basically telling the slave either “I want to read what you have in your address” or “I want to put information into your address. If there is a slave matching the address sent, the slave of that address with send the signal on the SDA bus at point “A” which means “acknowledge” that it is a real address and will pull the signal low. If the address does not exist, the signal will remain high.

**Read: The slave now sends data on the SDA line to the master of the contents of it**

**Write: The master will again send data on the SDA line to the slave for it to store**

The SCL line is just telling the slave to “sample” the data being sent which basically means taking the analog waves, and turning them into digital signals of 1’s and 0’s

*Note: This process only works if there is ONE register, multiple register will be explored below.*

![image6](https://user-images.githubusercontent.com/60119461/97534402-4fd0c880-1990-11eb-837a-310f6e756d5b.png)


Some devices (like the one we will be working with in the application section) will have multiple registers (Little pockets 8-bits of memory within the device.)The process of reading and writing to the salve will be a little different when dealing with this case.

 First, we will send the slave device's address along the SDA along with a write bit (or write function) ,then, we will send the register address. After we will send the slave device's address one more time and then we will either send a read or write bit depending on what we wish to do at that register(the picture up top illustrates reading a resister). After, the slave sends back the data stored within it’s register or stores data we have sent it within it’s register.

### Additional Resources

Concepts of I2C:

[https://www.youtube.com/watch?v=jFtr0Ha5f-c](https://www.youtube.com/watch?v=jFtr0Ha5f-c)

[https://www.youtube.com/watch?v=fm13tIe5wSc](https://www.youtube.com/watch?v=fm13tIe5wSc)

Application of I2C:

[https://www.youtube.com/watch?v=cvmQNTVJrzs](https://www.youtube.com/watch?v=cvmQNTVJrzs)

(Note: Some bit manipulation in this video is different than I will be explaining in the Application section. After reading through the comments, the video maker forgot to realize that the functions he was using already did bit-manipulation for him and he was just doing extra work. Other than that this is a very good video.)


# Application

To apply this I2C theory we will develop hardware and code that will read data from a device's register. The device we will be using to read a register from is the BNO055 which is a sensor that contains a gyroscope, accelerometer, magnetometer, and a temperature sensor. For this project we will read the die temperature sensor (heat generated by the device due to its operation) from the device. While the addresses of the registers needed to be access and how to use the data found within them have already been stated within the code, if you are new to I2C you should still take a look at the datasheet of the BNO055 [https://cdn-shop.adafruit.com/datasheets/BST_BNO055_DS000_12.pdf](https://cdn-shop.adafruit.com/datasheets/BST_BNO055_DS000_12.pdf) . This will help you better understand how some of the code operates, give you a better understanding  overall, help you understanding of reading datasheets, and help with future projects involving I2C.

### STM32

Note: if you are unfamiliar with STM32CUBEMX and/or how to operate it, please refer to the “Blink Program” documentation and it will outline step by step how to use it to program a MCU.

Open STM32CUBEMX and search for the microcontroller. Start a new project and follow the same setup steps outlined in the “Blink Program” documentation including the check off “Debug Serial Wire”. 

Now, click the on pins 62(PB9) and 61(PB8) and set their state to I2C1_SDA and I2C1_SCL respectively. Click the Connectivity then select I2C1 and change the drop-down menu from “Disable” to “I2C.” Configure the hardware connect the voltage, ground, SCL, and SDA between the MCU and the Sensor.

### HARDWARE

Similar to what we saw in the section of this document explaining what I2C is, we need to establish a SCL line and a SDA line that connects both the MCU master, and the BNO055 slave.

The components needed will be 10 jumper wires, a breadboard, an ST-Link, the MCU, and the BNO055. First, the ST-Link must be correctly wired. There should be 4 wires attached to the SWCLK, SWDI, GND, and 3.3V pins on the device.

![image7](https://user-images.githubusercontent.com/60119461/97534509-7f7fd080-1990-11eb-9fbc-61a3ace90ab5.png)


Next, the MCU should be placed along the middle of the breadboard at the top. The jumper wires connected to the 3.3V and GND must be connected to the power and ground busses on the breadboard respectively. The SWIDO wire should be connected to pin 46 and the SWCLK wire should be connected to pin 49 on the MCU.

![image8](https://user-images.githubusercontent.com/60119461/97534533-8c042900-1990-11eb-82e1-7ba313ba25d5.png)


Place one wire on the ground bus and one on the power bus. Connect the wire on the power bus to the 64 pin and the one on the ground bus to the 63 pin.

![image9](https://user-images.githubusercontent.com/60119461/97534565-9a524500-1990-11eb-86fd-cfa787e30119.png)


To power the BNO055 connect a jumper wire from it’s GND pin to the ground bus and it’s 3vo pin to the power bus.

![image10](https://user-images.githubusercontent.com/60119461/97534629-b5bd5000-1990-11eb-82a4-f3b73ab48655.png)


Now, to establish I2C communication we must connect the SDA and SCL of the two devices. Since in STM32 we established our SDA to be pin 62 MCU we connect that to the SDA pin on the BNO055. Similarly, since we established our SCL to be pin 61 MCU we connect that to the SCL pin on the BNO055. Normally we would also need to calculate a pull-up resistor and place it on the SCL and SDA bus so they would not become hanging. Thankfully, the BNO055 device has built in pull-up resistors so the hardware portion of this project should be relatively simple.

![image11](https://user-images.githubusercontent.com/60119461/97534739-e56c5800-1990-11eb-8f01-5d139fa66550.png)

![image12](https://user-images.githubusercontent.com/60119461/97534752-eac9a280-1990-11eb-94a3-1acea288fae5.png)

Since the bread board has a break in the power and ground buses, connect the break of the 2 buses. (Black and green wires in the picture).

![image13](https://user-images.githubusercontent.com/60119461/97534783-f917be80-1990-11eb-8729-d6eb3176ba42.png)

The result should look like the image below.

![image14](https://user-images.githubusercontent.com/60119461/97534824-0a60cb00-1991-11eb-823e-ff1333da58c7.png)



### REGISTERS

First what we need to do is find out what registers we will be writing to. After reading the datasheet it becomes obvious the registers needed will be:

![image15](https://user-images.githubusercontent.com/60119461/97534861-18aee700-1991-11eb-9ca5-dfa9c4757835.png)

PWR_MODE has the register address of 0x3E and controls the state the BNO055 device is sensor data.

![image16](https://user-images.githubusercontent.com/60119461/97534898-29f7f380-1991-11eb-8179-de8e362d1a96.png)


This is a list of all possible modes the device can be in, normal mode turns all sensors on, low power just has the accelerometer on and turns on after 5 seconds of no movement, and suspend mode where no sensors are on. The Reg Value just means what need to be written to the register in order to place it in that mode. Similarly, if that value is read from that address it means the device is in that mode. The x’s are bits that do not matter if they are 0’s or 1’s, all that matter are the non-x bits. For example, if I wanted to place the device into the Low Power Mode, I could write 00000001 or 00100001 or 11010101 or 11111101. If the last 2 bits stay 01 then it stays in Low Power Mode. Now potentially, the device could be using the other bits for something else so instead of directly writing a value to it, the better idea would be to read the current value within the address, use bit manipulation to get it into the xxxxxx01 form of Low Power Mode, and then return the new data to the address. We will discuss more about how to do bit manipulation later.

![image17](https://user-images.githubusercontent.com/60119461/97534930-37ad7900-1991-11eb-83b3-91d067606256.png)

OPR_MODE has the register address of 0x3D and is another register which controls which sensors will be in use but unlike PWR_MODE, it will not change unless the user does it themselves.

![image18](https://user-images.githubusercontent.com/60119461/97534958-41cf7780-1991-11eb-845e-f625cbe47d57.png)

![image19](https://user-images.githubusercontent.com/60119461/97534995-514ec080-1991-11eb-9823-de94c91fff05.png)


These two tables show which mode turns on which sensor and what the data in the register needs to be for it to be in that operating mode.

![image20](https://user-images.githubusercontent.com/60119461/97535045-64fa2700-1991-11eb-8f8e-0a96b8ae21d2.png)

TEMP has the register address of 0x34 and is the place we will read our temperature value from.

![image21](https://user-images.githubusercontent.com/60119461/97535076-6fb4bc00-1991-11eb-9661-9340c7f9d300.png)

UNIT_SEL has a register address of 0x3B and controls the units the data will be outputted in. This register doesn’t just hold the data for temperature units, but all other sensors too. Due to this we will only focus on changing the 5th bit. If the 5th bit is 0 it will display data for Celsius, and if it is 1 it will display data for Fahrenheit.

![image22](https://user-images.githubusercontent.com/60119461/97535103-7a6f5100-1991-11eb-8b90-f5223c647e7e.png)

This just means what we must divide our temperature value by to get the correct value. If its in Celsius it’s Temperature/1 aka just itself and if its in Fahrenheit, its Temperature/2. Note: LSB means Least Significant Bit and is the last bit in the data transmission for example in 10101111**0** the very last zero is the least significant bit.

Now that we know what registers we will be working with we can get to the actual code.



### CODE

There are 4 functions we will be using:

[**HAL_I2C_Master_Transmit**](https://os.mbed.com/users/EricLew/code/STM32L4xx_HAL_Driver/docs/tip/group__I2C__Exported__Functions__Group2.html#ga9440a306e25c7bd038cfa8619ec9a830) (I2C_HandleTypeDef *hi2c, uint16_t DevAddress, uint8_t *pData, uint16_t Size, uint32_t Timeout)

[**HAL_I2C_Master_Receive**](https://os.mbed.com/users/EricLew/code/STM32L4xx_HAL_Driver/docs/tip/group__I2C__Exported__Functions__Group2.html#ga6b3cef8c309e88ed6d3b8deba149aac9) (I2C_HandleTypeDef *hi2c, uint16_t DevAddress, uint8_t *pData, uint16_t Size, uint32_t Timeout)

[**HAL_I2C_Mem_Write**](https://os.mbed.com/users/EricLew/code/STM32L4xx_HAL_Driver/docs/tip/group__I2C__Exported__Functions__Group2.html#ga33e725a824eb672f9f999d9d5ce088fb) (I2C_HandleTypeDef *hi2c, uint16_t DevAddress, uint16_t MemAddress, uint16_t MemAddSize, uint8_t *pData, uint16_t Size, uint32_t Timeout)

[**HAL_I2C_Mem_Read**](https://os.mbed.com/users/EricLew/code/STM32L4xx_HAL_Driver/docs/tip/group__I2C__Exported__Functions__Group2.html#ga7b593a1b85bd989dd002ee209eae4ad2) (I2C_HandleTypeDef *hi2c, uint16_t DevAddress, uint16_t MemAddress, uint16_t MemAddSize, uint8_t *pData, uint16_t Size, uint32_t Timeout)

HAL_I2C_Master_Transmit() and HAL_I2C_Master_Receive() will actually be implemented within the code, but used to explain how HAL_I2C_Mem_Write() and HAL_I2C_Mem_Read() work.

What HAL_I2C_Master_Transmit() does is  it sends data on the SDA line for the slave to receive. While normally you would need a read/write bit, to indicate whether the slave will store this data, or send data back to that location this function takes care of it and automatically makes the last bit a write bit meaning the slave will store it.

*Note: If you are confused by what this means or how it would look please refer to “What is I2C” portion of this document, specifically how information travels on the SDA line (also if that doesn’t help please feel free to find videos online explaining it, I left a few resources within the document myself).*

Similar to Transmit, HAL_I2C_Master_Receive() has the master send the address of where it wants data to be stored to the SDA. Again normally we would have a read write bit but HAL_I2C_Master_Receive() automatically makes it a read bit so the slave sends data back on the SDA to the place in memory the pointer we gave the function is pointing to.

Now, since the BNO055 has multiple registers, we will need to call 2 functions, HAL_I2C_Master_Transmit() to send the address in memory we wish to read from in the BNO055, and then either HAL_I2C_Master_Transmit() again or HAL_I2C_Master_Receive() depending if we want to read that register or write to it.

What HAL_I2C_Mem_Write() and HAL_I2C_Mem_Read() do is make a shortcut by combining those operations, for example what HAL_I2C_Mem_Write() does is basically do the equivalency of calling HAL_I2C_Master_Transmit() twice, once to send the address of the register and once to send the data we want to write into the register. What HAL_I2C_Mem_Read() does is the equivalency of  calling HAL_I2C_Master_Transmit() to send the address of the register we want to read to from to the SDA, and then calling HAL_I2C_Master_Receive() to place the information of that register into some address in memory.

Understanding is the hardest part of I2C, the actual coding is relatively short.

First, we need to define the slave address of the BNO055. The data sheet says it is 0x28 in Low mode and 0x29 in high mode. The low and high mode just differ by a read and write bit so we can just use the 0x28 address instead. Since the device’s address only has 7 bits it must be bit shifted to the left by 1 giving us the new slave address of 0x50 making room for the extra read/write byte to be manipulated by our functions. Uint8_t just means it is an integer with 8 bits in it aka 1 byte, the size of the data stored in each register and the size of each register address.

    uint8_t slaveAddress = 0x50;
  
Next, we need to check if the Master is able to communicate with the slave using the function below. If there are any errors that cause, there to be the inability for communication the program with stop running.

    HAL_I2C_IsDeviceReady(&hi2c1,slaveAddress,1, HAL_MAX_DELAY);

Now, I define the places I want to read to write from with, it’s registers, data to be stored or sent, and a pointer to that data. While I could have used a struct when writing this code, I wanted to explicit as possible, but the same principle would apply as in the struct would have a register, data, and pointer to data and have 5 separate initializations of it, each corresponding to a different register.

    /*
     * Units
     * 5th bit is for temp sensor. 0 is for Celsius and 1 is for Fahrenheit 
     */
    		  uint8_t	unitsReg = 0x3B;
    		  uint8_t	UnitData=0;
    		  uint8_t *pUnits;
    		  pUnits=&UnitData;
    /*
     * Power
     */
    		  uint8_t	powerReg = 0x3E;
    		  uint8_t	powerData=0;
    		  uint8_t *pPower;
    		  pPower=&powerData;
    
    /*
     * Temperature
     */
    		  uint8_t tempdata = 0;
    		  uint8_t tempReg = 0x34;
    		  uint8_t *ptemp;
    		  ptemp=&tempdata;
    /*
     * Temperature Source
     * The temperature can either come from the gyroscope (xxxxxx01) or the accelerameter(xxxxxx00)
     */
    
    	      uint8_t sourceReg = 0x40;
    	      uint8_t sourceData = 0;
    		  uint8_t *pSource;
    		  pSource=&sourceData;

Next, I try to read from the power register. First call the HAL_I2C_Mem_Read() function passing it the parameters, the standard i2c handle, the slave address of BNO055, the register address for the Power, the size of the register (in this case its 8 bits aka 1 byte so I send 1), the pointer to where I wish to store the data located at the power register, how big the data I expect to receive is (in this case the register can only store 1 byte so I send 1), and the timeout.

After I have the data found in the device, I preform bit manipulation. I AND the data with the 11111100 in hexadecimal (I was having problems doing bit manipulation in binary for some reason) to put it into the power mode I wish aka normal (xxxxxx00).

With HAL_I2C_Mem_Write() I now send the new data back to the register as it takes the same parameters as HAL_I2C_Mem_Read(), but in this case pPower now points to the data I wish to write  into the register rather than store.

HAL_DELAY() just delays the function so data sent on the SDA bus isn’t being interfered with by other functions

Note: While the datasheet did say the BNO055 is in normal mode by default, I am just being explicit to give the best understanding to the reader. Changes in the other registers won’t be as redundant.

    //TURN ON POWER
    		HAL_I2C_Mem_Read(&hi2c1,slaveAddress,powerReg,1,pPower,1,50);
    		HAL_Delay(100);
    		//Bit manipulation to change the data the normal mode (goal is xxxxxx00)
    		powerData&=0xFC; //11111100
    		HAL_I2C_Mem_Write(&hi2c1,slaveAddress,powerReg,1,pPower,1,50);
    		HAL_Delay(100);

Following the same line of thinking for the other registers leads to the code below. First, we turn the power on by placing it to normal mode, turn all the sensors, change the place the sensor is read from to the accelerometer, put the units into Celsius, and read the operating temperature of the device.

    //TURN ON ALL SENSORS
    		HAL_I2C_Mem_Read(&hi2c1,slaveAddress,OPRreg,1,pOPR,1,50); //Checks what mode the sensor is in
    		HAL_Delay(100);
    		//Bit manipulation to change the data into the mode which allows all 3 sensors on (goal is xxxx0111)
    		OPRData&=0xF7;//11110111
    		OPRData|=0x07; //00000111
    		HAL_I2C_Mem_Write(&hi2c1,slaveAddress,OPRreg,1,pOPR,1,50);//Tells the sensors to turn on and collect data by changing mode
    		HAL_Delay(100);
    
    
    //TURN ON CHANGE TEMP READER TO ACCLEROMETER
    		HAL_I2C_Mem_Read(&hi2c1,slaveAddress,sourceReg,1,pSource,1,50);
    		HAL_Delay(100);
    		//Bit manipulation (goal is xxxxxx00)
    		sourceData&=0xFC; //11111100
    		HAL_I2C_Mem_Write(&hi2c1,slaveAddress,sourceReg,1,pSource,1,50);
    		HAL_Delay(100);
    
    
    //PUT UNITS INTO CELCIUS
    		HAL_I2C_Mem_Read(&hi2c1,slaveAddress,unitsReg,1,pUnits,1,50); //Checks to see the units
    		HAL_Delay(100);
    		//Bit manipulation to change the data into celcius while preserving data that is unrelated (goal is xxx0xxxx)
    		UnitData&=0xEF;//11101111
    		HAL_I2C_Mem_Write(&hi2c1,slaveAddress,unitsReg,1,pUnits,1,50);
    		HAL_Delay(100);
    
    
    //GET TEMP
    		HAL_I2C_Mem_Read(&hi2c1,slaveAddress,tempReg,1,ptemp,1,50);
    		HAL_Delay(100);

Since there is no easy way to print data in SMT32 we will have to follow the data within the debugger.


![image23](https://user-images.githubusercontent.com/60119461/97535338-e2259c00-1991-11eb-8407-cf9984b6f3f1.PNG)

Now, within the debug screen, in the bottom left corner, you should be able to still see the code you have written. Scroll down till reach the portion of the code which receives the temperature and just above it double click. This adds what is called a “break” and will allow us to skip to the code where we wish to actually look at what is happening with the data.

![image24](https://user-images.githubusercontent.com/60119461/97535357-e8b41380-1991-11eb-80ef-376e31cf5c44.PNG)



At the top, click the “continue” button till you reach the break point.

![image25](https://user-images.githubusercontent.com/60119461/97535425-03868800-1992-11eb-8383-e12b692c618d.PNG)



After, click the “step over” button to progress in the code line by line while cautiously monitoring the variables window in the top center for change.

![image26](https://user-images.githubusercontent.com/60119461/97535451-0b462c80-1992-11eb-8365-e9cc506a7e67.PNG)

The value read should be close to this, again, remember this is the operating temperature of the device and not the temperature of the room.

![image27](https://user-images.githubusercontent.com/60119461/97535568-36308080-1992-11eb-8d25-6bd80a028a4c.png)


