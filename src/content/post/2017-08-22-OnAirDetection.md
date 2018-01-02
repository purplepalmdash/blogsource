+++
title = "OnAirDetection"
date = "2017-08-22T14:32:51+08:00"
description = "OnAirDetection"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Direct Read From Serial Port
Arduino mega2560 Code:    

```
void setup() {
  pinMode(0,INPUT_PULLUP); 
  pinMode(1,INPUT_PULLUP); 
}

void loop() {
}
```
This code will turn mega2560 into a USB-TTL transmitter, thus you could
directly read from the serial port and display them in hex mode:    

```
# cat /dev/ttyACM0 | xxd -p -c 9
ffff01270005020534
ffff01270005020534
ffff01270005020534
ffff01270005020534
....
```
According to the reference manual, 

![/images/2017_08_22_14_36_13_1174x364.jpg](/images/2017_08_22_14_36_13_1174x364.jpg)

we know the air is 0.05 mg/m3.   

### Read From Arduino Mega2560
Code:    

```
void setup() {
  // initialize both serial ports:
  Serial.begin(9600);
  Serial2.begin(9600,SERIAL_8N1);

    while (!Serial) {
    ; // wait for serial port to connect. Needed for native USB port only
  }


    while (!Serial2) {
    ; // wait for serial port to connect. Needed for native USB port only
  }

  // prints title with ending line break
  Serial.println("ASCII Table ~ Character Map");
}

void loop() {


   for (int i=0; i<9; i++) {

    while(!Serial2.available()); // wait for a character
    int incomingByte = Serial2.read();
    Serial.print(incomingByte,HEX);
    Serial.print(' ');

   }
   Serial.println();
}
``` 
Result:    

![/images/2017_08_22_15_11_30_368x349.jpg](/images/2017_08_22_15_11_30_368x349.jpg)

### Problem
How to detect the result? Start from FF FF?    

Should be 

![/images/2017_08_22_15_24_13_265x289.jpg](/images/2017_08_22_15_24_13_265x289.jpg)


### Fixed-Bug
Every time it will print starting with `FF FF`:    

```
int incomingByte=0;
int vanishByte=0;

void setup() {
  // initialize both serial ports:
  Serial.begin(9600);
  Serial2.begin(9600,SERIAL_8N1);

    while (!Serial) {
    ; // wait for serial port to connect. Needed for native USB port only
  }


    while (!Serial2) {
    ; // wait for serial port to connect. Needed for native USB port only
  }

  // prints title with ending line break
  Serial.println("ASCII Table ~ Character Map");
}

void loop() {

    //Serial.println();
   //for (int i=0; i<9; i++) {
    while(!Serial2.available()); // wait for a character
    vanishByte = incomingByte;
    incomingByte = Serial2.read();
    if((incomingByte == 0xFF) && (vanishByte != 0xFF))
    {
      Serial.println();
    }
    Serial.print(incomingByte,HEX);
    Serial.print(' ');
   //}
   //Serial.println();
}
```
![/images/2017_08_22_15_24_13_265x289.jpg](/images/2017_08_22_15_24_13_265x289.jpg)
