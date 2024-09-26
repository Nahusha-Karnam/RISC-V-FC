# RISC-V Flight Computer
## Overview
We are using the VSD Squadron Mini which uses the 32-bit CH32V003F4U6 microcontroller based on RISC-V architecture as the Primary Flight Computer

## Features
- VSD Squadron Mini as the primary flight computer
- NRF24L01 PA+LNA for near range radio communication
- Arduino Nano as the communication interface between NRF24L01 PA+LNA and VSD Squadron Mini (Arduinon nRF doesn't support RISC-V boards a new library is in development)

## Flight Computer
![image](https://github.com/user-attachments/assets/427a126a-f084-4e91-9138-f149d6d50837)

## Flight Controller
![image](https://github.com/user-attachments/assets/245508f1-bbec-4bb0-a404-9577720980e9)

## Ground Level Testing
https://github.com/user-attachments/assets/71c4f411-5ce5-4add-9790-7fed913bb83c

## RISC-V Flight Computer Code
```
int ch_1=0;
int ch_2=0;
int ch_3=0;
int ch_4=0;
int ch_5=0;
int ch_6=0;

struct Signal {
  byte throttle;
  byte pitch;
  byte roll;
  byte yaw;
  byte aux1;
  byte aux2;
};

Signal data={0,0,0,0,0,0};

void setup()
{
  Serial.begin(9600);
  Serial.setTimeout(200); 
  
} 
void writeMicroseconds(int servopin,int ms){
  pinMode(servopin,OUTPUT);
  digitalWrite(servopin,HIGH);
  delayMicroseconds(ms);
  digitalWrite(servopin,LOW);
  delayMicroseconds(2500-ms);
}
void loop()
{
 
  if (Serial.available() >= sizeof(data)) {
    Serial.readBytes((char*)&data, sizeof(data));
    ch_1 = data.throttle;
    ch_2 = data.pitch;
    ch_3 = data.roll;
    ch_4 = data.yaw;
    ch_5 = data.aux1;
    ch_6 = data.aux2;
    writeMicroseconds(PD2,ch_1*10);
    writeMicroseconds(PC4,ch_2*10);
    writeMicroseconds(PC3,ch_3*10);
    writeMicroseconds(PC0,ch_4*10);
    Serial.print("Throttle: ");
    Serial.print(ch_1);
    Serial.print(" Pitch: ");
    Serial.print(ch_2);
    Serial.print(" Roll: ");
    Serial.print(ch_3);
    Serial.print(" Yaw: ");
    Serial.print(ch_4);
    Serial.print(" Aux1: ");
    Serial.print(ch_5);
    Serial.print(" Aux2: ");
    Serial.println(ch_6);
  }
} 
```

