20180324
### Dual Core Audio Framework for ESP32

Purpose: The ESP-32 has Analog To Digital (ADC) and Digital To Analog (DAC) converters built-in. Could these on-Board devices be used in place of external analog interface peripherals (typically with i2s or spi interfaces) to reduce system cost/complexity in voice audio applications?

For the purpose of these demos, circuits external to the ESP32 were restricted to analog signal conditioning components only.

###### Applications
The scope of this demonstration is limited to voice audio (**<4KHz**). Applications in the voice audio range include: game audio, voice notifications, voice grade telephone service or radio communications circuits, walkie-talkie/intercom/baby monitor/surveillance type apps, voice compression and processing, vocoders, baseband signaling and audio test signals.

This ESP32 based audio framework uses the popular Arduino IDE and could be used to teach Digital Signal Processing concepts. At a university or in a workshop at a technical conference or maker class, it could be used as a low cost hands-on lab for students. Time wouldn't be wasted with setup or teaching the build tools.


###### Findings
Proof of concept demos programs were created and tested over a one month period.
* The 8-bit DAC limits the dynamic range to 48 dB and limits fidelity but may be adequate for some voice applications.
* The ADC is not driven directly by a hardware sample clock but is sampled in software on a timer interrupt creating sample clock skew. This results in some phase noise on the sampled audio.
* The sample service core processor is only loaded at 13% at the 8Ksps rate and could be increased to a higher sample rate.
* A frame rate processing technique used in the demos (v.s. sample-by-sample processing) introduces latency--audio input to output.
* The ring buffers in FreeRTOS were too slow to operate at the test Frame Rate (128 msec., needed 256 msec. )
* The demos were compatible with concurrent  Bluetooth Low Energy operation. WiFi not checked.

###### Proof Of Concept Demos:
Dual core concurrent multi-task processing of continuous real-time audio
on the ESP-32.

Tested on Espressif ESP32 Dev board
Rev. 1 silicon

Arduino IDE with the ESP32 core from espressif

ADC/DAC sample processing at 8Ksps for voice audio range (< 4KHz).
Not suitable for music as is.

Core 1 -- Sample Service

Core 0 -- Application Processor

Cores use alternating ping-pong buffers to pass samples between them every
N samples. Tasks synchronize on a "FrameAvailable" semaphore.

Frame processing
runs every N samples on the Application Processor. The Frame processing technique
introduces latency delay (audio input to output).

Concurrency examples include:
* Continuous sine wave
* Dual tone (frequencies re-create old telephone dial tone)
* Continuous phase frequency sweep test signal (300--3000 Hz in 5 sec.,repeated)
* Microphone talk-through
* Pink Noise example from portaudio.com
* Voice Alert Example
__________________________________________________________________



bobolink
twitter: @wm6h

__________________________________________________________________

ref:
esp32 arduino ide
https://github.com/espressif/arduino-esp32

Sensorsslot's ESP32 Arduino IDE Dual core multi-tasking intro:
YouTube video: https://www.youtube.com/watch?v=k_D_Qu0cgu8
 http://esp32.info/docs/esp_idf/html/dd/d3c/group__xTaskCreate.html

some watchdog info:
 https://github.com/espressif/arduino-esp32/issues/595

You can do Digital Filtering with a MCU
YouTube https://www.youtube.com/watch?v=k0LHp6jtHA8

High-Voltage Signal Conditioning for Low-Voltage ADCs
http://www.ti.com/lit/an/sboa097b/sboa097b.pdf

Audio Out Circuit.

The PAM 8403 uses 5VDC from the ESP-32 Dev Board.
Output is on DAC1 GPIO pin 25 of the ESP32
No DAC re-construction filter is used.
Some aliasing on the sweep app is heard at high volume. Maybe a re-construction
at the DAC output would help.

Audio In circuit: SparkFun Electret Microphone Breakout BOB-12758
https://www.sparkfun.com/products/12758
It has a freq. response to 10KHz and we sample at 8Ksps so a pole is added to
its response to turn it into a 1st order (20 dB/Decade power drop-off)
anti-aliasing filter (LPF) with 4KHz cutoff
Add a 47pF cap across R5 (820K). From JP1-3 to U1-4 is easier.
We would like a 2nd order filter here (40 dB/Decade power drop-off).
 https://cdn.sparkfun.com/datasheets/BreakoutBoards/Electret_Microphone_Breakout_v20.pdf

note: this circuit could be avoided by sampling at 16Ksps


mic audio is on analogRead(34)
ADC_WIDTH_9Bit to more or less match the DAC 8 bits.
An ADC_ATTEN_11db pad is set in order to run the mic at 3.3 Volts

3/10/18 Pink Noise example from portaudio.com modified for ESP32 Audio Framework. Statistics haven't been verified with spectrum analyzer.

3/15/18 modified sine generator with nkolban's BLE library. Select freq of sine over BLE (30-3000 Hz UTF-8
unsigned int string < 5 chars).   https://youtu.be/MGNU7COFlT4

3/21/18 Voice Alert example. Audio samples stored as an array in the source. File edited in Audacity (Audacity.com) and saved as mono, 8Ksps, signed 16-bit PCM Microsoft WAV file. Converted to C source with xxd -i fn.wav fn.c

![BlockDiagram](/ESP32AudioFrameworkBlock.png)
# ESP32_AudioFramework
