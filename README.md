# README: Arduino Color Sensor Project

## Overview
This project utilizes an Arduino microcontroller and a color sensor to detect the color of an object and display its name in English on an LCD screen. It is an engaging way to learn about interfacing sensors with Arduino and displaying data visually.

## Features
- Detects the displayed color of an object using a color sensor.
- Displays the name of the detected color on an LCD screen in English.
- Provides real-time feedback for the user.

## Components Required
1. Arduino Uno R3 board 
2. Color sensor module (TCS3200)
3. LCD screen (16x2 LCD)
4. Breadboard and jumper wires
5. Power supply or USB cable for Arduino
6. Resistors 

## Circuit Diagram
1. Connect the color sensor module to the Arduino:
   - VCC to 5v
   - GND to GND
   - SCL to Arduino SCL (A5 on Uno)
   - SDA to Arduino SDA (A4 on Uno)

2. Connect the LCD screen to the Arduino:
   - For I2C LCDs:
     - VCC to 5V
     - GND to GND
     - SCL to Arduino SCL (A5 on Uno)
     - SDA to Arduino SDA (A4 on Uno)
   - For standard LCDs:
     - Refer to the pinout of your specific LCD and wire accordingly.

## Installation
1. Install the required libraries in the Arduino IDE:
   - For the color sensor:
     - TCS3200: Install "TCS3200" or similar library
   - For the LCD:
     - Standard LCD: Use "LiquidCrystal" library

2. Upload the Arduino sketch to your board using the Arduino IDE.

## Usage
1. Place an object in front of the color sensor.
2. The sensor will detect the color and the Arduino will process the data.
3. The name of the detected color will appear on the LCD screen.

## Code Explanation
Below is the Arduino code used for this project:

```cpp
#include <LiquidCrystal.h> //Include the LCD library 

//Define lcd and corresponding pins
LiquidCrystal lcd(7, 8, 9, 10, 12, 13);

// define pins
const int S0 = 2;
const int S1 = 3;
const int S2 = 4;
const int S3 = 5;
const int signal = 6;

// define variables for pulses
unsigned long Red;
unsigned long Blue;
unsigned long Green;
unsigned long clear;

// define char string variable to store colour
const char *colors[] = {"Invalid distance", "Black", "Red", "Blue", "Green", "White", "Yellow", "Purple", "Orange"};
unsigned int c = 0;

void setup() { 
  // set pin modes
  pinMode(S0,OUTPUT);
  pinMode(S1,OUTPUT);
  pinMode(S2,OUTPUT);
  pinMode(S3,OUTPUT);
  pinMode(signal,INPUT);

  // Set up LCD size and PWM to output right amount of contrast for displaying 
  analogWrite(11, 50);  //Traditionally lcd contrast is controlled by a 10k potentiometer, however ours was not functional and we did not have enough time to get a new one
  lcd.begin(16, 2);     //Thus pin 11, a PWM pin, was used to manually set the contrast in setup and provided proper LCD functionality
 
  /* set frequency scaling - 
     S0 S1 | Output frequency scaling
     L  L  | power down
     L  H  | 2%
     H  L  | 20%
     H  H  | 100%
  */
  digitalWrite(S0,HIGH);
  digitalWrite(S1,LOW);

  // initialize serial communication
  Serial.begin(9600);
}

void loop() { 
 
  GetColors();                                     //Execute the GetColors() function to get the value of each RGB color sensor

  /* map the red, green, and blue values to a more intuitive 0-255 range where
     0 means less light (Darker colours) and 255 means more (Lighter colours). This part will require calibration
     depending on your colored surfaces and ambient light levels.
  */
  Red = map(Red,160,20,0,255);
  Green = map(Green,160,20,0,255);
  Blue = map(Blue,160,20,0,255);

  if (Red <= 35 && Green <= 60 && Blue <= 70){         //If the values are low it's likely the black color (little colour being reflected)
    c = 1;                                        //Sets colours to Black
}
  else if (Red>Blue && Red>Green && Red>= 150 && Green <= 60 && Blue <= 100){      //if Red value is the largest one and RBG values within expected ranges
    c = 2;                                        //Sets colors to Red

  }
  else if (Blue>Green && Blue>Red && Blue>170 && Red < 80 && Green < 160){    //Same thing for Blue
    c = 3;                                        //Sets colors to Blue

  }
  else if (Green>Red && Green>Blue && Green > 90 && Red <= 55 && Blue <= 75){           //Same thing for Green
    c = 4;                                        //Sets colors to Green

  }
  else if (Red >= 200 && Red <= 255 && Green >= 200 && Green <= 255 && Blue >= 200 && Blue <= 255){ //White was the simplest as it is the only colour that puts all RGB values over 200
    c = 5;                                        //Sets colors to White
  }


    else if (Red <= 230 && Red > Green && Green > Blue && Blue >= 120){ //Same as the Red, Green, and Blue
    c = 6;                                        //Sets colors to Yellow
  }

    else if (Blue <= 135 && Blue > Red && Red > Green && Green >= 45){ //Same as the Red, Green, and Blue
    c = 7;                                        //Sets colors to Purple 
  }

    else if (Red <= 215 && Red > Green && Green > Blue && Blue <= 135 && Green <= 150){  //Conditions for Orange were a little trickier due 
    c = 8;                                        //Sets colors to Orange                //to the similarity with yellow in RGB comparisons
  }
  else{
     c = 0;                  //if the color is not recognized, set color to Invalid distance 
}

  delay(1000);                                   //1s delay 

  // print readings, used extensively for testing and configuration
  
  
//  Serial.print("Red: ");
//  Serial.print(Red);
//  Serial.print(" | Green: ");
//  Serial.print(Green);
//  Serial.print(" | Blue: ");
//  Serial.println(Blue);
  
lcd_output();

  delay(1000);                                   //1s delay to be able to see the output on lcd without changing too quickly

}

void lcd_output(){

// Setup cursor location and clear screen before printing new output
lcd.setCursor(0, 0);
lcd.clear();

//Checks to make sure c value is within range of the String list
if (c < sizeof(colors) / sizeof(colors[0])) {
    lcd.print(colors[c]);
} else {
    lcd.print("Unknown");
}

}


void GetColors(){

  /* cycle through each filter type and use the pulseIn command to measure pulse length.
  Frequency *increases* with more light, so the pulse length will *decrease*.
  /  S2 S3 | Photodiode Type
     L  L  | Red
     L  H  | Blue
     H  L  | Clear (no filter)
     H  H  | Green
  */
  
  // clear, unused but conveinient for traversing the different LOW/HIGH pairings
  digitalWrite(S2,HIGH);
  digitalWrite(S3,LOW);
  clear = pulseIn(signal,HIGH);
  
  // red
  digitalWrite(S2,LOW);
  digitalWrite(S3,LOW);
  Red = pulseIn(signal,HIGH);

  // green
  digitalWrite(S2,HIGH);
  digitalWrite(S3,HIGH);
  Green = pulseIn(signal,HIGH);

  // blue
  digitalWrite(S2,LOW);
  digitalWrite(S3,HIGH);
  Blue = pulseIn(signal,HIGH);

}
```

## Example Output
When a red object is placed in front of the sensor, the LCD screen will display:
```
Detected Color:
Red
```

## Troubleshooting
1. **No Output on LCD**: Ensure the LCD connections and I2C address are correct.
2. **Incorrect Color Detection**: Adjust the sensor's position or calibration.
3. **Code Errors**: Verify all required libraries are installed and up to date.

## Customization
- Add more color names or refine the detection logic by modifying the RGB-to-color mapping in the code.
- Integrate additional sensors or modules for enhanced functionality.

## License
This project is open-source and available under the MIT License. Feel free to modify and share it.

## Acknowledgments
- Adafruit for their TCS34725 library and tutorials.
- Arduino community for their continuous support and resources.

---

Enjoy building and customizing your Arduino Color Sensor Project!

