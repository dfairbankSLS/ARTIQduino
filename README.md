# ARTIQduino
This project describes the basic hardware and code to fully operate a trapped ion optical clock using an Arduino Mega. It is based on the gra-afch AD9959 shield. We jokingly started referring to it as "ARTIQ"-duino as it can perform similar tasks as the popular ARTIQ platform, but with a price tag under $300. Note that this has no afliliation with the professional grade ARTIQ plaform, I developed and tested this as a side project while at Colorado State University and later at Stable Laser Systems.

**Link to AD9959 Shield:** https://gra-afch.com/catalog/rf-units/dds-ad9959-arduino-shield-rf-signal-generator-600-mhz-1-5-ghz-core-clock-low-spurs-low-harmonic/

## Capabilities:
- Four DDS channels with frequency control up to 225MHz with better than 1us on/off amplitude modulation for pulse sequences (via AOMs).
- Hardware counter for counting PMT counts during detect pulse (ArduinoMega TCNT5, pin 47)
- Plenty of DIO/TTL channels for additional on/off control for pulse sequences
- Analog inputs and various other features on the Arduino are generally available
- Broad scans over cooling transition, calibration for uniform optical power vs frequency, rf tickle experiments to find secular mode frequencies, clock transition frequency scans, Rabi flopping, clock-style two point probes.
- Simple control via serial communications (For example via Jupyter Notebook etc.), and Arduino code can be extended for customizable functionality.

## Limitations:
- In the current implementation, the min frequency resolution is 1Hz.
- Assuming double pass configuration, the max frequency shift you can apply is about 450MHz
- Typially a single AOM is used for near-detuned cooling and detection, which requires adjusting the DDS frequency between the two operations. The speed of this is limited by the Ardunio clock speed and currently takes about 100us. This is uniform between pulses and typically isn't an issue unless you have very high heating rates (such that that 100us delay would allow excessive heating).
- The RF output power directly from the shield is only about -7dBm max, so you need about a 35dB RF amplifier for each AOM channel.
  **Suitable RF Amp, WYDZ-PM-1M-750MHz-3W** https://www.ebay.com/itm/364304389209, consider getting one or two extra in case one gets toasted.
- PMT counting is limited by Arduino clock speed, can't be faster than 1/4 the ardunio clock speed (16MHz). So, if you have pulses coming in faster than 1/(4MHz) = 250ns you will miss some counts. This typically isn't a problem since in say a 250us detection pulse maybe you're getting somethign like 10 counts (only 4kHz on average).

## Other notes:
- The amplitude modulation is applied to the DDS simply by toggling digital pins on the Arduino Mega which are connected to DDS pins P0,P1,P2,P3. This switches between whatever amplitude is set in two different amplitde registers(so I typically have one register set to zero amplitude and the other to whatever amplitude I need to drive the AOM at, for on/off behavior). This makes it extremely easy to program.
- PMT counts are collected on TCNT5 which is hardwiared to ArduinoMega pin 47. All you have to do is clear TCNT5 prior to the detect pulse and read out after. There is no processing time or interrupts required.
