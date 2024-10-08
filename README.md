# RISC-V Flight Computer
## Overview
We are using the VSD Squadron Mini which uses the 32-bit CH32V003F4U6 microcontroller based on RISC-V architecture as the Primary Flight Computer

## Features
- VSD Squadron Mini as the primary flight computer
- NRF24L01 PA+LNA for near range radio communication
- Arduino Nano as the communication interface between NRF24L01 PA+LNA and VSD Squadron Mini (Arduinon nRF doesn't support RISC-V boards a new library is in development)

## Flight Computer
![image](https://github.com/user-attachments/assets/427a126a-f084-4e91-9138-f149d6d50837)

## Flight Computer Circuit Diagram
<img width="732" alt="vsdfc" src="https://github.com/user-attachments/assets/9ff52c79-7744-4288-a4a6-03a394ed5dfc">

## Flight Controller
![image](https://github.com/user-attachments/assets/7c03af38-7ebb-418e-9b0a-a36e91db5e99)

## Flight Controller Circuit Diagram
![image](https://github.com/user-attachments/assets/245508f1-bbec-4bb0-a404-9577720980e9)

## Demo at SEMICON India 2024

https://github.com/user-attachments/assets/76631395-3a9d-4301-8ceb-1479f3b206da


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
## Receiver Arduino Code
```
#include <SPI.h>
#include <nRF24L01.h>
#include <RF24.h>

int ch_1 = 0;
int ch_2 = 150;
int ch_3 = 150;
int ch_4 = 150;
int ch_5 = 0;
int ch_6 = 0;



struct Signal {
byte throttle;      
byte pitch;
byte roll;
byte yaw;
byte aux1;
byte aux2;
};

Signal data;

const uint64_t pipeIn = 0xE9E8F0F0E1LL;
RF24 radio(2, 3); 

void ResetData()
{
  data.throttle = 100;   
  data.pitch = 150;   
  data.roll = 150;     
  data.yaw = 150;     
  data.aux1 = 0;   
  data.aux2 = 0; 
}

void setup()
{
  //Set the pins for each PWM signal 
 

  //Configure the NRF24 module
  ResetData();
  radio.begin();
  radio.openReadingPipe(1,pipeIn);
  radio.setAutoAck(false);
  radio.setDataRate(RF24_250KBPS);
  radio.setPALevel(RF24_PA_MIN);
  radio.startListening(); //start the radio comunication for receiver
  Serial.begin(9600);

}
unsigned long lastRecvTime = 0;

void recvData()
{
while ( radio.available() ) {
radio.read(&data, sizeof(Signal));
lastRecvTime = millis(); 
  }
    }
void loop()
{
recvData();
unsigned long now = millis();
if ( now - lastRecvTime > 1000 ) {
ResetData(); 
}
ch_1 = data.throttle;     // pin D5 (PWM signal)
ch_2 = data.pitch;     // pin D3 (PWM signal)
ch_3 = data.roll;     // pin D4 (PWM signal)
ch_4 = data.yaw;    // pin D2 (PWM signal)

ch_5 = data.aux1;     // pin D6 (PWM signal)
ch_6 = data.aux2;     // pin D7 (PWM signal)


Serial.write((byte*)&data, sizeof(data));
delay(100);
}
```
## Transmitter Code
```
#include <SPI.h>
#include <nRF24L01.h>
#include <RF24.h>
  const uint64_t pipeOut = 0xE9E8F0F0E1LL;  
  RF24 radio(2, 3); // select CE,CSN pin 
  struct Signal {
  byte throttle;
  byte pitch;
  byte roll;
  byte yaw;
  byte aux1;
  byte aux2;
};
  Signal data;
  void ResetData() 
{
  data.throttle = 0;  
  data.pitch = 150;   
  data.roll = 150;     
  data.yaw = 150;   
  data.aux1 = 150;    
  data.aux2 = 150;    
}
  void setup()
{
  //Start everything up
  radio.begin();
  radio.openWritingPipe(pipeOut);
  radio.setAutoAck(false);
  radio.setDataRate(RF24_250KBPS);
  radio.setPALevel(RF24_PA_MIN);
  radio.stopListening(); //start the radio comunication for Transmitter | Verici olarak sinyal iletişimi başlatılıyor
  ResetData();
 Serial.begin(9600);
}

  void loop()
{

  data.roll = map( analogRead(A3), 0, 1023, 100 ,200 );
  data.throttle = map(analogRead(A0),1023,1,100,200);
  data.pitch = map(analogRead(A1),0,1023,100,200);
  data.yaw = map(analogRead(A6),0,1023,100,200);
  Serial.println(data.roll);
  radio.write(&data, sizeof(Signal));
}
```
