# Flipdot_NTP_Clock
Flipdot clock that updates time from an NTP server

# Overview


# Hardware

# Software

# RS485
## Protocol
## Data Frame Structure
Below is an example of a data frame structure for a flip dot with ONE controller.
| Header | Command | Address | Display Data | End |
| :---: | :---: | :---: | :---: | :---: |
| 1 byte | 1 byte | 1 byte | 28 or 112 bytes | 1 byte |
| 0x80 | See Command | See Address | See Display Data | 0x8F |

Below is an example of a data frame structure for a flip dot with TWO controllers.
| Header | Command | Address | Display Data | Header | Command | Address | Display Data | End |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| 1 byte | 1 byte | 1 byte | 28 or 112 bytes | 1 byte | 1 byte | 1 byte | 28 or 112 bytes | 1 byte |
| 0x80 | See Command | See Address | See Display Data | 0x80 | See Command | See Address | See Display Data | 0x8F |


### Command Byte
The purpose of the command byte is to let the controller know when to refresh the display.
There are multiple different commands that can be used for various displays and system requirements.
Pay attention to the # of data bytes for the command.
0x82 is a blank command that updates the display without a data byte number requirement.
For example, a 14 by 28 pixel display with two controllers can use the 28 data byte command to update half of the display at a time or the 56 to update to entire display at once.
| Command in Hex | Command in Binary | # of Data Bytes | # of Controllers | Display Size | Refresh |
| :---: | :---: | :---: | :---: | :---: | :---: |
| 0x81 | B10000001 | 112 | One | Four 7x28 Displays | No |
| 0x82 | B10000010 | NA | NA | NA | Yes |
| 0x83 | B10000011 | 28 | Two | One 14x28 Display | Yes |
| 0x84 | B10000100 | 28 | Two | One 14x28 Display | No |
| 0x85 | B10000101 | 56 | Two | One 14x28 Display | Yes |
| 0x86 | B10000110 | 56 | Two | One 14x28 Display | No |

### Address Byte
The address byte correlates to the different controllers and displays.
Address byte values range anywhere from 0x00 to 0xFF in hex.
In order for the display to work properly the DIP switch on the back of each controller must be set in binary to match desired address.
For example, if the address DIP switch from 1-6 for one display is set to 011111 and the other display to 000001, the respective addresses are 0x1F and 0x01

### Data Bytes
Data is sent one byte (8 bits) at a time.
Flipdot displays come in various shapes and sizes but all that I have come accross have rows that are split into groupings of 7.
For example, my flipdot display is 14 by 28 pixels which is actually two displays put together (2x 7 by 28 pixels).
The 7 pixels in each row are programmed by one byte. To program a 14 by 28 pixel display 56 (28*2) bytes of data are required.
I find using binary numbers is best for the data programming in order to get a visual of the display while coding but hex will work as well.
Since there are only 7 pixels to program but 8 bits in a byte, the most significant bit can be a 1 or a 0.
Below is an example of the flipdot output for a given byte.
| B7 | B6 | B5 | B4 | B3 | B2 | B1 | B0 |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| 0 or 1 | 1 | 1 | 0 | 1 | 1 | 1 | 0 |
| NA | White | White | Black | White | White | White | Black |

# ESP32
## Protocol
## NTP Server

# Code
```text
/****************************************************************************    
 *    Project: Flip Dot NTP Clock                                           *
 *    Written by: Tristan Weger                                             *
 *    Date: 02/11/2023                                                      *
 *    Description: The following code uses an ESP32 wifi module             *
 *      to connect to a NTP server to obtain local time and update          *
 *      a clock made from a 14x28 pixel Alfazeta XY5 flip dot display.      *   
 *      This code can be retrofit for any size flip dot display.            *
 ****************************************************************************/

#include <WiFi.h>
#include "time.h"
#include <Wire.h>
#include <TimeLib.h>

const char* ssid     = "Wifi SSID Here";
const char* password = "Wifi Password Here";

//Different NTP servers are available this one sets to the best signal
//See GitHub repo for daylight savings and time zone offset setting
const char* ntpServer = "pool.ntp.org";
const long  gmtOffset_sec = -18000;
const int   daylightOffset_sec = 3600;

//Head and end are format in hex as a standard
byte HCA1[] = {0x80, 0x85, 0x01};
byte HCA2[] = {0x80, 0x85, 0x02};
byte ender[] = {0x8F};

//Display bytes are format in binary for visualization
byte topRow[] = {
  B00000000
  };
byte zero[] = {
  B00011100,
  B00100010,
  B00100010,
  B00100010,
  B00100010,
  B00100010,
  B00100010,
  B00011100,
  B00000000
  };
byte one[] = {
  B00001000,
  B00011000,
  B00001000,
  B00001000,
  B00001000,
  B00001000,
  B00001000,
  B00011100,
  B00000000
  };
byte two[] = {
  B00011100,
  B00100010,
  B00000010,
  B00000100,
  B00001000,
  B00010000,
  B00100000,
  B00111110,
  B00000000
  };
byte three[] = {
  B00011100,
  B00100010,
  B00000010,
  B00001100,
  B00000010,
  B00000010,
  B00100010,
  B00011100,
  B00000000
  };
byte four[] = {
  B00000100,
  B00001100,
  B00010100,
  B00100100,
  B00111110,
  B00000100,
  B00000100,
  B00000100,
  B00000000
  };
byte five[] = {
  B00111110,
  B00100000,
  B00100000,
  B00111100,
  B00000010,
  B00000010,
  B00100010,
  B00011100,
  B00000000
  };
byte six[] = {
  B00001100,
  B00010000,
  B00100000,
  B00111100,
  B00100010,
  B00100010,
  B00100010,
  B00011100,
  B00000000
  };
byte seven[] = {
  B00111110,
  B00000010,
  B00000100,
  B00000100,
  B00001000,
  B00001000,
  B00010000,
  B00010000,
  B00000000
  };
byte eight[] = {
  B00011100,
  B00100010,
  B00100010,
  B00011100,
  B00100010,
  B00100010,
  B00100010,
  B00011100,
  B00000000
  };
byte nine[] = {
  B00011100,
  B00100010,
  B00100010,
  B00100010,
  B00011110,
  B00000010,
  B00000100,
  B00011000,
  B00000000
  };
  
void displayNumber(int n) {
    switch(n){
        case 0:
            Serial.write(zero, 9);
            break;
        case 1:
            Serial.write(one, 9);
            break;
        case 2:
            Serial.write(two, 9);
            break;
        case 3:
            Serial.write(three, 9);
            break;
        case 4:
            Serial.write(four, 9);
            break;
        case 5:
            Serial.write(five, 9);
            break;
        case 6:
            Serial.write(six, 9);
            break;
        case 7:
            Serial.write(seven, 9);
            break;
        case 8:
            Serial.write(eight, 9);
            break;
        case 9:
            Serial.write(nine, 9);
            break;
        default:
            Serial.write(zero, 9);        
    }
    return; 
}

byte onesS, tensS, onesM, tensM, onesH, tensH;

void setup(){
  Serial.begin(57600);
  Serial.println();
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    //Waiting to connect to wifi, display symbol on flipdot as wanted
  }
  configTime(gmtOffset_sec, daylightOffset_sec, ntpServer);
  flipDotDisplayTime();
  WiFi.disconnect(true);
  WiFi.mode(WIFI_OFF);
}

void loop(){
  flipDotDisplayTime();
  delay(1000);
}

void flipDotDisplayTime(){
  struct tm timeinfo;
  if(!getLocalTime(&timeinfo)){
    //Failed to retrieve time, display warning on flipdot as wanted
    return;
  }
  int setHour = timeinfo.tm_hour;
  int setMinute = timeinfo.tm_min;
  int setSecond = timeinfo.tm_sec;
  //12 hour display, remove for 24 hour
  if (setHour>12){
    setHour = setHour - 12;
  }else if (setHour == 0){
    setHour = 12;
  }
  //Splits seconds into tens and ones digit for clock
  tensS = (setSecond/10);
  tensS = tensS-((tensS/10)*10);
  onesS = setSecond-((setSecond/10)*10);
  //Splits minutes into tens and ones digit for clock
  tensM = (setMinute/10);
  tensM = tensM-((tensM/10)*10);
  onesM = setMinute-((setMinute/10)*10);
  //Splits hours into tens and ones digit for clock
  tensH = (setHour/10);
  tensH = tensH-((tensH/10)*10);
  onesH = setHour-((setHour/10)*10);

  Serial.write(HCA1, 3);
  Serial.write(topRow, 1);
  displayNumber(tensH);
  displayNumber(tensM);
  displayNumber(tensS);
  Serial.write(HCA2, 3);
  Serial.write(topRow, 1);
  displayNumber(onesH);
  displayNumber(onesM);
  displayNumber(onesS);
  Serial.write(ender, 1);
  Serial.println();
}
```
