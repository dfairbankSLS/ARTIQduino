# ARTIQduino
This project describes the basic hardware and code to operate a trapped ion optical clock using an Arduino Mega. It is based on the gra-afch AD9959 shield. We jokingly started referring to it as "ARTIQ"-duino as it can perform similar tasks as the popular ARTIQ platform, but with a price tag under $300. Note that this has no afliliation with the professional grade ARTIQ plaform, I developed and tested this as a side project while as a graduate student at Colorado State University and then later at Stable Laser Systems.

**Link to AD9959 Shield:** https://gra-afch.com/catalog/rf-units/dds-ad9959-arduino-shield-rf-signal-generator-600-mhz-1-5-ghz-core-clock-low-spurs-low-harmonic/

## Capabilities:
- Four DDS channels with frequency control up to 225MHz with sub-1us timing precision in amplitude modulation for pulse sequences (via AOMs).
- Hardware counter for counting PMT counts during detect pulse (ArduinoMega TCNT5, pin 47)
- Plenty of DIO/TTL channels for additional on/off control for pulse sequences
- Analog inputs and various other features on the Arduino are generally available
- Broad scans over cooling transition, calibration for uniform optical power vs frequency, rf tickle experiments to find secular mode frequencies, clock transition frequency scans, Rabi flopping, clock-style two point probes.
- Simple control via serial communications (For example via Jupyter Notebook etc.), and Arduino code can be extended for customizable functionality.

## Limitations:
- In the current implementation, the minimum DDS frequency resolution is 1Hz. The minimum pulse duration is 4us. Pulse duration can be set in 1us increments, but the pulse length is repeatable and precise.
- Assuming double pass AOM configuration, the max frequency shift you can apply is about 450MHz.
- Typially a single AOM is used for both near-detuned cooling and detection, which requires adjusting the DDS frequency and amplitude between the two operations. The speed of this is limited by the Ardunio clock speed and currently takes about 100us. This is uniform between pulses and typically isn't an issue unless you have very high heating rates (such that that 100us delay would allow excessive heating).
- The RF output power directly from the shield is only about -7dBm max, so you need about a 35dB RF amplifier for each AOM channel.

  **Suitable RF Amp, WYDZ-PM-1M-750MHz-3W** https://www.ebay.com/itm/364304389209, consider getting one or two extra in case one gets toasted.
- PMT counting is limited by Arduino clock speed, can't be faster than 1/4 of the 16MHz ardunio clock speed. So, if you have pulses coming in faster than 1/(4MHz) = 250ns you will miss some counts. This typically isn't a problem since in say a 250us detection pulse maybe you're expecting something like 10 counts (only 4kHz on average).

## Additional Hardware:
- One ion trap and associated lasers and vacuum hardware.
- A arduinoMega screw terminal breakout board is helpful to make additional DIO/analog connections (~$10 on Amazon).
- For ultra-stable clock laser and any accessory lasers (cooling, repump, clearout, ablation etc.) feel free to contact us at SLS (https://stablelasers.com/contact/, dfairbank@stablelasers.com). We have experience with ion traps and associated lasers (especially for Be+, Ca+, Sr+).
- We also have separate professional grade pulse generation and ion trap control and automation electronics in development (not using the Arduino or AD9959 shield), please inquire for details.

## Code Installation:
- In general install the same way as instructed for the gra-afch shield, this code is just a modified version. Install Arduino IDE and copy the libraries into the libraries section of the installed Arduino directory. Note that the encoder library is modified from the normal so Arduino will complain. Mostly just need to install the Ardunio code, compile and upload to the ArduinoMega. The serial baud rate used here is 115200.
  
- There are two main code sections that have been adjusted DDS_firmware.ino and ReadSerialCommands.ino, so I will summarize the changes:
  
DDS_firware.ino changes:
  1. The P0-P3 pins for modulation functions of the DDS are connected to pins 16,15,14,4 of the Arduino, respectively so these have been defined such that you can address the pins as "PO_PIN", "P1_PIN" etc. The hardware counter for PMT pulses is conencted to Ardino pin 47, so this is defined to address as "COUNTER_PIN".
  2. InitCounter5() function has been added to initialize for counting pulses on TCNT5 of the ATmega2560.
  3. enableAmplitudeModulation() function added to make sure the settings for two level amplitude modulation are set up. This function was added to the AD9959 library if you want to adjust or change it (and so also note AD9959 library has been modified from default downloaded from gra-afch, to have this function).
  4. In setup the default on/off state for each channel when powering on is set. You can change this to whatever you prefer. Note that in this implementation 0 is ON and 1 is OFF for the DDS!

ReadSerialCommands.ino changes:
- This library was completely rewritten to accept and parse serial commands of the form "command(arg0,arg1,arg2,...)". All the pulse sequencing etc. is programmed here. Basic list of commands:
- coolDetect(uint32_t detectFrequency, uint16_t detectAmp, uint32_t duration, uint16_t reps) do reps of coolDetect pulse sequence, returns average PMT counts detected
- clockRabi(detFrq,detAmp,detDur,clkFrq,clkAmp,clkDur,spFrq,spAmp,spDur,reps) do a clock pulse sequence FD cool, ND cool, clock pulse then detect
- setFD/ND(uint32_t frequency, int16_t amplitude_dBm, uint32_t duration) set the pulse parameters for far deturned or near detuned cooling in dBm
- setND_ASF(uint32_t frequency, int16_t amplitude, uint32_t duration) set near detuned parameters, amplitude in ASF 0-1023
- setCL(uint32_t frequency, int16_t amplitude) set clock channel default parameters (DDS channel 2)
- setIR(uint32_t frequency, int16_t amplitude) set IR channel default parameters (DDS channel 3)
- ablate(uint16_t reps) I used this to send TTL pulse for ablation loading on Arduino pin 49
- on/off(Channel 0-3) turn on channel Px
- saveToEEPROM() -- frequency and amplitude settings remain on power cycle
- h — Help

One thing to be aware is the gra-afch code was set up to have the amplitude adjusted in dBm, but for faster operation I have in some cases switched to just setting in the 10bit (0-1023) format. The jupyter notebook code has functions to convert back aand forth from one to the other.

## Hardware connections:
- Far Detuned AOM I have on DDS channel 0
- Near Detuned/Detect AOM on DDS channel 1
- Clock AOM or rf-tickle I put on DDS channel 2
- IR repump (ie for Ca+ or Sr+) I had on DDS channel 3
- PMT should be connected to ArduinoMega pin 47

## Other notes:
- The amplitude modulation is applied to the DDS simply by toggling digital pins on the Arduino Mega which are connected to DDS pins P0,P1,P2,P3. When configured for two level amplitude modulation, each switches between whatever amplitude is set in two different amplitde registers for each separate channel (so I typically have one register set to zero amplitude and the other to whatever amplitude I need to drive the AOM at on a given channel). This pin toeggling makes it extremely easy to program pulse sequences.
- PMT counts are collected on ATMega2560 TCNT5 which is hardwired to ArduinoMega pin 47. All you have to do is clear TCNT5 prior to the detect pulse and read out after. There is no processing time or interrupts required.
- Good luck and happy trapping!

## Example Images:
Simple USB serial interface (Python, Jupyter Notebook, etc.)
![Interface](https://github.com/user-attachments/assets/9bdf5700-b32c-455a-bd56-039009737e5f)

Ten consecutive cool and detect pulse sequences
![10 Pulses](https://github.com/user-attachments/assets/f8fd1a68-4909-46f5-9ced-7af050718a18)

Example clock pulse sequence with IR repump AOM channel.
![Clock Probe](https://github.com/user-attachments/assets/58885b00-1e2d-4020-bbfb-9b81d31cb006)

Scan over dipole allowed cooling transition
![cooling scan](https://github.com/user-attachments/assets/a63c9e40-055c-43a8-b718-46b271915da0)

Rf tickle probe to determine radial secular mode frequencies.
![radial rf tickle](https://github.com/user-attachments/assets/060799d4-13d8-41ee-a23f-aeabf9fd7b4e)

Example Rabi Flops on a clock transition
![flop 1](https://github.com/user-attachments/assets/fab18d0b-428f-415c-9b9b-146806a2b37c)
![flop 2](https://github.com/user-attachments/assets/ebc8bafe-4d2f-4c53-9efc-9cd7210fe5d3)
