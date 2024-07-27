# ARTIQduino
This project describes the basic hardware and code to fully operate a trapped ion optical clock using an Arduino Mega. It is based on the gra-afch AD9959 shield. We jokingly started referring to it as "ARTIQ"-duino as it can perform similar tasks as the popular ARTIQ platform, but with a price tag under $300. Note that this has no afliliation with the professional grade ARTIQ plaform, I developed and tested this as a side project while as a graduate student at Colorado State University and the later at Stable Laser Systems.

**Link to AD9959 Shield:** https://gra-afch.com/catalog/rf-units/dds-ad9959-arduino-shield-rf-signal-generator-600-mhz-1-5-ghz-core-clock-low-spurs-low-harmonic/

## Capabilities:
- Four DDS channels with frequency control up to 225MHz with sub-1us timing precision in amplitude modulation for pulse sequences (via AOMs).
- Hardware counter for counting PMT counts during detect pulse (ArduinoMega TCNT5, pin 47)
- Plenty of DIO/TTL channels for additional on/off control for pulse sequences
- Analog inputs and various other features on the Arduino are generally available
- Broad scans over cooling transition, calibration for uniform optical power vs frequency, rf tickle experiments to find secular mode frequencies, clock transition frequency scans, Rabi flopping, clock-style two point probes.
- Simple control via serial communications (For example via Jupyter Notebook etc.), and Arduino code can be extended for customizable functionality.

## Limitations:
- In the current implementation, the min DDS frequency resolution is 1Hz and minimum pulse duration is 4us.
- Assuming double pass AOM configuration, the max frequency shift you can apply is about 450MHz.
- Typially a single AOM is used for both near-detuned cooling and detection, which requires adjusting the DDS frequency between the two operations. The speed of this is limited by the Ardunio clock speed and currently takes about 100us. This is uniform between pulses and typically isn't an issue unless you have very high heating rates (such that that 100us delay would allow excessive heating).
- The RF output power directly from the shield is only about -7dBm max, so you need about a 35dB RF amplifier for each AOM channel.

  **Suitable RF Amp, WYDZ-PM-1M-750MHz-3W** https://www.ebay.com/itm/364304389209, consider getting one or two extra in case one gets toasted.
- PMT counting is limited by Arduino clock speed, can't be faster than 1/4 of the 16MHz ardunio clock speed. So, if you have pulses coming in faster than 1/(4MHz) = 250ns you will miss some counts. This typically isn't a problem since in say a 250us detection pulse maybe you're expecting something like 10 counts (only 4kHz on average).

## Additional Hardware Required:
- One ion trap and associated lasers and vacuum hardware.
- For ultra-stable clock laser and any accessory lasers (cooling, repump, clearout, ablation etc.) feel free to contact us at SLS (https://stablelasers.com/contact/, dfairbank@stablelasers.com). We have experience with ion traps and associated lasers (especially for Be+, Ca+, Sr+).
- If you're looking to take the performance the next level but maintain an intuitive and simple serial interface, we have in development a professional grade rack mount electronic box for pulse sequence generation and ion trap control / automation. One notable feature is amplified DDS output with convenient amplitude control like the AD9910 but frequency resolution which can match the AD9912.

## Other notes:
- The amplitude modulation is applied to the DDS simply by toggling digital pins on the Arduino Mega which are connected to DDS pins P0,P1,P2,P3. This switches between whatever amplitude is set in two different amplitde registers(so I typically have one register set to zero amplitude and the other to whatever amplitude I need to drive the AOM at, for on/off behavior). This makes it extremely easy to program.
- PMT counts are collected on TCNT5 which is hardwired to ArduinoMega pin 47. All you have to do is clear TCNT5 prior to the detect pulse and read out after. There is no processing time or interrupts required.

## Example Images:
![Interface](https://github.com/user-attachments/assets/9bdf5700-b32c-455a-bd56-039009737e5f)
![10 Pulses](https://github.com/user-attachments/assets/f8fd1a68-4909-46f5-9ced-7af050718a18)
![Clock Probe](https://github.com/user-attachments/assets/58885b00-1e2d-4020-bbfb-9b81d31cb006)
![cooling scan](https://github.com/user-attachments/assets/a63c9e40-055c-43a8-b718-46b271915da0)
![radial rf tickle](https://github.com/user-attachments/assets/060799d4-13d8-41ee-a23f-aeabf9fd7b4e)
![flop 1](https://github.com/user-attachments/assets/fab18d0b-428f-415c-9b9b-146806a2b37c)
![flop 2](https://github.com/user-attachments/assets/ebc8bafe-4d2f-4c53-9efc-9cd7210fe5d3)
