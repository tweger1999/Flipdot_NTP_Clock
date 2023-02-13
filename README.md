# Flipdot_NTP_Clock
Flipdot clock that updates time from an NTP server

# Overview


# Hardware

# Software

# RS485
## Protocol
## Data Structure


| Header | Command | Address | Display Data | End |
| :---: | :---: | :---: | :---: | :---: |
| 1 byte | 1 byte | 1 byte | 28-112 bytes | 1 byte |
| 0x80 | See Command | See Address | See Display Data | 0x8F |


### Commands
| Command in Hex | Command in Binary | # of Data Bytes | # of Controllers | Display Size | Refresh |
| :---: | :---: | :---: | :---: | :---: | :---: |
| 0x81 | B10000001 | 112 | One | x4 7x28 | No |
| 0x81 | B10000010 | NA | NA | NA | Yes |
| 0x83 | B10000011 | 28 | Two | 14x28 | Yes |
| 0x84 | B10000100 | 28 | Two | 14x28 | No |
| 0x85 | B10000101 | 56 | Two | 14x28 | Yes |
| 0x86 | B10000110 | 56 | Two | 14x28 | No |

### Address

### Data Byte
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

