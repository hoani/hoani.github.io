---
title: "Arduino Cheatsheet"
excerpt: "Basic arduino cheatsheet"
permalink: /guides/engineering/electronics/arduino-cheatsheet
toc: true
categories:
  - guide
  - electronics
---

## Timing

Duration timers, starts at `0` at boot.
```cpp
uint32_t ms = millis();
uint32_t us = micros();
```
Delays
```cpp
delay(uint32_t ms);
delayMicroseconds(uint32_t us);
```

## Serial

Boards may have more than one serial channel, `Serial` is the primary; then `Serial1`, `Serial2`, etc.

Enable a serial channel with `baud` rate:
```cpp
Serial.begin(int baud);
if (Serial) // = true when Serial is ready
```

Receiving bytes:
```cpp
int numBytes = Serial.avaliable();
int byte = Serial.read(); // returns -1 if no byte avaliable
size_t numBytesRead = Serial.readBytes(char* buffer, int len);
```

Writing bytes:
```cpp
Serial.write(int byte);     // write a single byte
Serial.write(char* string); // write a \0 term string
Serial.write(char* buffer, int len);
```

Print writing:
```cpp
Serial.print(int val);
Serial.print(int val, HEX); // BIN, OCT, DEC, HEX
Serial.print(float val);
Serial.print(float val, int decimalPlaces);
```

Close serial port:
```cpp
Serial.end();
```

## Digital I/O

Setting pin modes
```cpp
pinMode(int pin, OUTPUT);
pinMode(int pin, INPUT);
pinMode(int pin, INPUT_PULLUP);
```

Using digital pins:
```cpp
digitalWrite(int pin, LOW);
digitalWrite(int pin, HIGH);
```
```cpp
int val = digitalRead(int pin); // return is HIGH/LOW
```

### Analog

Resolution capability depends on the board being used (Teensy 3.2 can be changed from 2 to 13 bits).

Default is 10-bits. 

```cpp
pinMode(int pin, INPUT);
analogReadResolution(int pin, int resolution);
uint16_t val = analogRead(int pin);
```

### PWM

Resolution capability depends on the board being used (Teensy 3.2 can be changed from 2 to 16 bits).

Default is 10-bit PWM.

```cpp
pinMode(int pin, OUTPUT);
analogWriteResolution(int pin, int resolution);
analogWrite(int pin, int val);
```

If your board has DAC support, the same code works.
