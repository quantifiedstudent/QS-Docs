# E66 smartwatch protocol

Author: Neal Geilen and Jelle Maas
Date of writing: September 20th, 2021

## Table of Contents

[[toc]]

## Context

For the Quantified Student Watch project we need to have proper communication with a smartwatch. After a research done by Ferris Kwaijtaal, he discovered that we can communicate with the E66 smartwatch over a Bluetooth LE connection.

The next step is finding out how to communicate with the E66 smartwatch over the Bluetooth link. Ferris Kwaijtaal already did some research on this topic. However, we found that his research was incomplete in some areas. For this reason, we are continuing the reverse engineering of the Bluetooth protocol of the E66 smartwatch. The results of this reverse engineering process will be reported in this document.

## Bluetooth Low Energy

The E66 smartwatch communicates over Bluetooth Low Energy (also known as BTLE). Bluetooth Low Energy is independent of classic [Bluetooth](https://en.wikipedia.org/wiki/Bluetooth_BR/EDR "Bluetooth BR/EDR") and has no compatibility, but BR/EDR and LE can coexist.

### Characteristics

Characteristics are items of data which relate to a particular internal state of the device or perhaps some state of the environment which the device can measure using a sensor. The current battery level is an example of internal state data whereas the ambient temperature could perhaps be measured by a sensor.

The UUID of the service and characteristic used for communication with the E66 smartwatch is: `be940001-7333-be46-b7ae-689e71722bd5`.

## Packet format

A packet consists of five parts, the command type, address, length, data and checksum.

| #  | Description    | Type                | Length  |
| -- | -------------- | ------------------- | ------- |
| P1 | Command type   | Unsigned byte       | 1 byte  |
| P2 | Data length    | Unsigned short      | 2 byte  |
| P3 | Data           | Unsigned byte array | * bytes |
| P4 | CRC16 checksum | Unsigned short      | 2 bytes |

### CRC16 checksum

The [CRC](https://en.wikipedia.org/wiki/Cyclic_redundancy_check)16 checksum is calculated using the _CRC-16/CCITT-FALSE_ algorithm, which uses `0x1021` as the polynomial, `0xffff` as the initial value and `0x0000` as the final XOR value.

Underneath a small snippet for calculating the CRC16 checksum in [Rust](https://www.rust-lang.org/):

```rust
fn crc16(data: &Vec<u8>) -> u16 {
    let mut crc: u16 = 0xffff;
    for b in data.iter() {
        crc ^= (*b as u16) << 8;
        for _ in 0..8 {
            if crc & 0x8000 != 0 {
                crc = (crc << 1) ^ 0x1021;
            } else {
                crc = crc << 1;
            }
        }
    }
    return crc;
}
```

## Commands

The E66 has several commands, during the reverse engineering process, we discovered the commands listed below.

| #       | Description                    | Command byte array |
| ------- | ------------------------------ | ------------------ |
| C1      | Close real time step           | 3 9 0 0 1          |
| C2      | Open real time step            | 3 9 1 0 1          |
| C3      | History sport data             | 5 2 1              |
| C4      | History HR data                | 5 6 1              |
| C5      | Respiratory rate data          | 5 9 1              |
| C6      | Delete sport history data      | 5 64 2             |
| C7      | Delete sleep history data      | 5 65 2             |
| C8      | Delete heart history data      | 5 66 2             |
| C9      | Delete blood history data      | 5 67 2             |
| C10     | Delete respiratory data        | 5 68 2             |
| C11     | Delete sport history data file | 5 64 3             |
| C12     | Delete sleep history data file | 5 65 3             |
| C13     | Delete heart history data file | 5 66 3             |
| C14     | Delete blood history data file | 5 67 3             |
| C15     | Delete respiratory data file   | 5 68 3             |
| **C16** | Write EGC                      | 3 11 1 1           |
| **C17** | Get history data               | 5 9 1              |
| **C18** | Get electricity                | 2 0 71 67          |
| C19     | Get device screen size         | 2 11               |
| C20     | Recover to default             | 1 14 82 83 89 83   |
| C21     | Get base info                  | 2 0 71 67          |
| C22     | Get support function           | 2 1 71 70          |
| **C23** | Get mac                        | 2 2 71 77          |
| C24     | Get device notification status | 2 4 71 83          |
| **C25** | Get device name or version     | 2 3 71 80          |
| **C26** | Get current heart rate         | 2 5 72 82          |
| **C27** | Get current BP                 | 2 6 66 80          |

The bold commands, are of most importance to the Delta watch project.
