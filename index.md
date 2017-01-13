My wanderings into the world of Mechatronics
============================================

Introduction
------------

As a pure Mechanical engineer I never had much time for electrical systems and
their ilk. It was considered anathema to the very core concept of "mechanical"
engineering. But my roommate at University worked in robotics and he
occasionally needed help when he was swamped. I always wanted to learn how to
solder, so I watched a couple youtube videos and off I went. Got an Arduino
starter kit for my birthday - turned an LED on and off and never looked back; I
was addicted.

An interesting thing I discovered in my musings is that because I was learning
in an unstructured environment, i.e., I was teaching myself, I couldn’t be
trusted to remember everything. Right from outset, I was obsessed about
documenting everything I noticed. I wrote down all my projects in a little lab
book. My internal policy became that I should be able to leave my projects for
years and know exactly what I did and why I did it when I checked my notes. But
then I moved cities, and my book was misplaced in the ensuing chaos. I’ve been
away from this tinkering for over a year now. Thus this is an erstwhile
continuation of my dear departed notes. This is, after all, the digital age.

Recently I was given the task of building a small and simple motor controller
(not one of those advanced PID controllers). The task – Use a potentiometer to
change the speed and direction of a motor, including a microcontroller into the
loop.

Theory
------

After some time spent researching on the Internet and familiarizing myself with
the best way to accomplish the task, I decided to refresh my memories with the
theory of how MOSFETs and Transistors worked. In short they’re solid state
switches that can turned ‘on and off’ with a voltage signal. In technical
parlance they would act similar to SPST switches if a **HIGH/LOW** signal is
passed to their *Gate/Base*. But an advantage of these devices is that, like a
dimmer switch, they have varying values between “*On and Off*” allowing control
over the amount of current flowing through the switch.

There are two types of transistors: Bipolar Junction Transistors (**BJTs**) and
Field Effect Transistors (**FETs**).

### MOSFETs

The term MOSFET stands for Metal-Oxide-Semiconductor Field Effect Transistor. It
has three terminals that are known as *Gate*, *Source* and *Drain*

Equipment used
--------------

After some time spent researching on the Internet and familiarizing myself with
the best way to accomplish the task, I took stock of the equipment I had. The
traditional way for directional motor control was to use an H-Bridge. I didn’t
have one. But I knew that an H-Bridge effectively amounts to 4 MOSFETS (for a
single channel). As we can see below:

![Figure 1](http://www.bristolwatch.com/ele/moshbridge/mosh8.png "Standard H-Bridge with both n and p type MOSFETs")

You’ll note that it uses the following to act as 4 switches:

- 2 n-type MOSFETs
- 2 p-type MOSFETs
- 2 n-type Transistors

The transistors exist to isolate the microcontroller from the higher voltage and
motor noise. But, driving the pin high on the transistor opens the gate in the
nMOSFET and closes the gate in the pMOSFET at the same time. But I lacked a
pMOSFET, so I decided to disregard them for my particular purposes. I decided to
use 4 nMOSFETs and some programming to simulate the same effect. The circuit I
built is shown below:

Figure 2

Figure 2

##Code



```cpp
/* Control a motor direction and speed with a potentiometer using n-MOSFETS as an H Bridge */

#include <Streaming.h>

// Use these PWM pins to control the power MOSFETs
/* Atmega328p has Analogwrite on 3,5,6,9,10 and 11. PWM on most pins is 490 Hz
   except for 5 and 6 on the UNO which operate on 980 Hz. */
   #define PowerPin1  11
   #define PowerPin2  9
   #define PowerPin3  5
   #define PowerPin4  10
   #define PotReadPin A0
   #define EncdAPin 2
   #define CountsPerRot 512

// For changing a variable inside an interrupt use a volatile datatype
volatile long EncdPosCW = 0.0;
volatile long EncdPosCCW = 0.0;

// Initialising Global Variables
//long prevEncdPosCW = 0;
double rotationsCW = 0;
double prevrotationsCW = 0;
double rotationsCCW = 0;
double prevrotationsCCW = 0;
int direction = 0;


void setup()
{
  attachInterrupt(digitalPinToInterrupt(EncdAPin), chkEncdA, RISING); // Interrupt on DPin 2
  
  // Initialise Arduino UNO pins
  pinMode(PowerPin1, OUTPUT);
  pinMode(PowerPin2, OUTPUT);
  pinMode(PowerPin3, OUTPUT);
  pinMode(PowerPin4, OUTPUT);

  Serial.begin(9600); // Initialize serial port
  Serial.println("Serial Interface Initialized");
}

void loop()
{
  int MotorSpeed = 0;
  int dirandSpeed = ReadPotVal();
  
  // Serial.println(dirandSpeed); // Be careful with Serial.prints in this sketch it overloads the serial buffer
  // since there is no delay. But the UNO doesn't operate fast enought to overflow the buffer and crash it. 
  // Only the Arduino DUE(ARM) and possibly Leonardo (ATMega32u4) can cause crashes.
  
  if(dirandSpeed<455) // 512 is the halfway point of the potentiometer readout
  {
    MotorSpeed = map(dirandSpeed, 500, 0, 0, 255); // Leave 100 values in the middle as deadband
    CWRot(MotorSpeed);  
  }
  else if(dirandSpeed>555)
  {
    MotorSpeed = map(dirandSpeed, 524, 1024, 0, 255); // Leave 100 values in the middle as deadband
    CCWRot(MotorSpeed);
  }
  else
  {
    NoRot();
  }
  //Serial.println(EncdPos);
  rotationsCW = double(EncdPosCW)/4096;
  rotationsCCW = double(EncdPosCCW)/4096;
  printRots();
  prevrotationsCW = rotationsCW;
  prevrotationsCCW = rotationsCCW;
}

void CCWRot(int MotSpeedVal)
{
  analogWrite(PowerPin1, MotSpeedVal);
  digitalWrite(PowerPin2, LOW);
  digitalWrite(PowerPin3, LOW);
  digitalWrite(PowerPin4, HIGH);
  direction = 2;
}

void CWRot(int MotSpeedVal)
{
  digitalWrite(PowerPin1, LOW);
  analogWrite(PowerPin2, MotSpeedVal);
  digitalWrite(PowerPin3, HIGH);
  digitalWrite(PowerPin4, LOW);
  direction = 1;
}

void NoRot()
{
  digitalWrite(PowerPin1, LOW);
  digitalWrite(PowerPin2, LOW);
  digitalWrite(PowerPin3, LOW);
  digitalWrite(PowerPin4, LOW);
  direction = 0;
  //clearAndHome();
}

int ReadPotVal()
{
  int PotVal = 0;
  PotVal = analogRead(PotReadPin);
  return PotVal;
}

void chkEncdA()
{
  if(direction==1)
  {
    EncdPosCW+=1;
  }
  else if(direction==2)
  {
    EncdPosCCW+=1;
  }
}

void clearAndHome() 
{ 
  Serial.write(27); 
  Serial.print("[2J"); // clear screen 
  Serial.write(27); // ESC 
  Serial.print("[H"); // cursor to home 
} 

void printRots()
{
  if(prevrotationsCW != rotationsCW || prevrotationsCCW != rotationsCCW)
  {
      Serial << "CWRot: " << rotationsCW << " CCWRot: " << rotationsCCW << endl;
  }
}

void plannedObso()
{
  
}
```