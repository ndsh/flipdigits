# Flipdigits
Alfazeta Small 7-segment displays aka SSM 1" Display aka Flipdigits

## Technical data
|                   |                             |
|-------------------|-----------------------------|
| Supply voltage    | 19V - 24V DC                |
| Control Interface | RS485                       |
| Operating temp.   | -40°C to +70°C              |
| Humidity          | up to 95% (no condensation) |

## General Information
The controller is built to drive one 1” small seven segment module. It is using RS485 to communicate a master unit which can be any rs485 data sending device (a PC with RS485 interface, Arduino, Raspberry Pi, etc) Each digit has got its own unique address and communication speed. In order to set these parameters you need to use a protocol described below or use a dedicated software.

Address domains: 0 - 255 (hex notation: 0x00 to 0xFF)

### Communication speed
1. 1200 bit/s
2. 2400 bit/s
3. 4800 bit/s
4. 9600 bit/s
5. 19200 bit/s
6. 38400 bit/s
7. 57600 bit/s
8. 115200 bit/s

### IDC6 Connector
The IDC6 connector, normally used for ISP, is used to carry voltage and information through a ribbon cable.
A main microcontroller can provide serial data which is then transported to a network of Flipdigits, which can be interlocked with each other through 4-pin Edge connectors which exist on each side of the Flipdigit.

## Protocol

### Frame structure
| Header | command           | Address      | Data              | End byte |
|--------|-------------------|--------------|-------------------|----------|
| 0x80   | Description below | 0x00 - 0xFF* | Description below | 0x8F     |

### Data
MSB is neglected, the following bits B6 - B0 are setting dots from top to bottom respectively.

### Commands
| Command | Number of data bytes | Automatic refresh | Description                                                                                                                                                |
|---------|----------------------|-------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 0x89    | 1                    | Yes               | Send one data byte and show it on the display                                                                                                              |
| 0x8A    | 1                    | No                | Send one data byte and wait for refresh                                                                                                                    |
| 0x8B    | 1                    | -                 | Set RS speed: 0x00 – 1200bit/s 0x01 – 2400bit/s 0x02 – 4800bit/s 0x03 – 9600bit/s 0x04 – 19200bit/s 0x05 – 38400bit/s 0x06 – 57600bit/s 0x07 – 115200bit/s |
| 0x8C    | 1                    | -                 | Set address: 0x00 – 0xFE (0xFF broadcast)                                                                                                                  |
| 0xC0    | 0                    | -                 | Format 0x80, 0xC0, 0x8F  –  request for address, response  –  0xAA, 0xCC, [address]      

### Defaults
Default settings out of the box is a RS speed of 9600bit/s and address set to 255 (0xFF).

## Example
Here is one super easy example for your Arduino with the help of a Max485 Module. (Schematics will come later)

First we send our header by saying this is a frame (0x80), then put the command (0x89) at Addr. 0xFF (controller at address 255 or broadcast).
Then we send our data, a byte (0, 1 or 2).
Last we close the frame to end transmission.

Done. Hello Flipdigit!

```c
byte frameHeader[]= {0x80, 0x89, 0xFF};
byte frameClosure[]= {0x8F};

void setup() {
  Serial.begin(57600);  
}

void loop() {
  Serial.write(frameHeader, 3);
  Serial.write(0);
  Serial.write(frameClosure, 1);
  delay(500);
  
  Serial.write(frameHeader, 3);
  Serial.write(1);
  Serial.write(frameClosure, 1);
  delay(500);
  
  Serial.write(frameHeader, 3);
  Serial.write(2);
  Serial.write(frameClosure, 1);
  delay(500);
}


```
