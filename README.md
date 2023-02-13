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

