# Index
This folder contains relevant datasheet for the sun sensors currently in development for the CAN-SBX balloon project, and which will later be used for the full satellite. The following is a description of each file, and relevant info on the part and it's uses.

### BOB-09056_MuxBreakout.pdf
  * Basic description of the analog multiplexer breakout board.
  * Will be used to select which photodiode should be read by the satellite. Greatly reduces number of pins required for measurement readings.

### <s>BPD-BQDA34-1.pdf</s>
  * Full datasheet of the photodiode.
  * Note, that this photodiode is no longer being used for development.

### cd74hct4067_Mux.pdf
  * Full datasheet of the chip used in the multiplexer.
  * Contains useful information on electrical characteristics and the channel selection truth table.

### lf411.pdf
  * Full datasheet of the operational amplifier.
  * Used as a transimpedance amplifier which is necessary to read voltages off photodiodes.

### lm555.pdf
  * Full datasheet of the 555 timer.
  * Used as an oscillator in a negative voltage generator. The negative voltage is required to provide symmetric power to the opamp.

### <s>LTR-323DB.pdf</s>
  * Full datasheet of the photodiode.
  * Note, that this photodiode is no longer being used for development.

### SFH_2430.pdf
  * Full datasheet of the photodiode.
  * This is the photodiode we are currently using in the sunsensor array.
  * Contains all necessary electrical and physical characteristics of the photodiode.
    * Note that while directional characteristics are provided in the datasheet, it is recommended that each sensor is individually calibrated.
