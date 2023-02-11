# Flipdot_NTP_Clock
Flipdot clock that updates time from an NTP server

# Overview


# Hardware

# Software

# RS485
## Protocol
## Data Structure


| Header | Command | Address | Display Data | End |
| --- | --- | --- | --- | --- |
| 1 byte | 1 byte | 1 byte | 28-112 bytes | 1 byte |
| 0x80 | 0x81-0x86 | 0x00-0xFF | 0x00-0xFF | 0x8F |

# ESP32
## Protocol
## NTP Server

