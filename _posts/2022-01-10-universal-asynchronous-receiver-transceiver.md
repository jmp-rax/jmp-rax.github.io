---
layout: post
title: "Universal Asynchronous Receiver Transceiver"
date: 2022-01-10 23:22:00 -0400
categories: UART embedded protocol note
---

> This article is a personal note I took to help remember the gist of UART

# UART

**Universal Asynchronous Receiver/Transmitter**

```
           1 0 1 0 1 0 1 0
Tx ‾‾‾|_|‾|_|‾|_|‾|_|‾|_|‾‾‾‾ Rx
       ^ start bit       ^^ stop bit
```

| Field | Summary |
| --- | --- |
| Wires | 2   |
| Speed | 9600, 19200, 38400, 57600, 115200, 230400, 460800, 921600, 1000000, 1500000 |
| Methods of Transmission | Asynchronous |
| Maximum Number of Masters | 1   |
| Maximum Number of Slaves | 1   |

## Overview

> The UART interface does not use a clock signal to synchronize the transmitter and receiver devices; it transmits data asynchronously. Instead of a clock signal, the transmitter generates a bitstream based on its clock signal while the receiver is using its internal clock signal to sample the incoming data. The point of synchronization is managed by having the same baud rate on both devices. Failure to do so may affect the timing of sending and receiving data that can cause discrepancies during data handling. The allowable difference of baud rate is up to 10% before the timing of bits gets too far off.
> 
> <cite>https://www.analog.com/en/analog-dialogue/articles/uart-a-hardware-communication-protocol.html</cite>

## Clock Cycle

The clock cycle is defined as *1 / baud rate*. The clock for both the sender and receiver need to be close enough for the duration of the conversation such that the drift doesnt affect the conversation. [CITE]

Example

> 1/115200 HZ = 8.681 micro seconds

## Start Bit

In UART the data transmission line is typically held at a high voltage level. When data is ready to be sent, the transmitter will pull the data transmission line low (ground) to signal the start of a UART Frame for one clock cycle.

Be careful not to assume the first bit part of the data frame. Logic analyzers will sometimes struggle here and cause frame errors in the capture.

## Data Frame

Data frames can be between 5-8 bits long (9 if the parity bit is not used), In most cases the data is sent as LSB or least significant bit.

## Parity

Parity describes the evenness or oddness of a number. It can be used as a simple check to determine if any data changed in transmission. If the total of the message is even the parity bit is set to 0, if the total is odd it is set to 1.

On the receiving end the receiver performs the same check to determine if the message changed in transmission.

## Stop Bit

The transmission line is driven high for one to two clock cycles indicating the transmission has ended. The receiver will also know the transmission ended because the data frame is a fixed length. This will help detect overrun and underrun errors.