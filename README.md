# home-assistant-hanbus
Home Assistant component for modbus devices with large registers


## Overview

This project begins as a fork of the modbus built-in integration from Home Assistant core. The scope here is to provide an
integration eclectic enough to cover the customizations and extensions to the original protocol, while leveraging the custom
data type, allowing any sequence of bytes in the modbus payload to be converted to a Python struct.

This project aims to bring back the ability to handle registers of arbitrary length, and potentially other features that 
may be in violation of the original Modbus protocol, but which are de facto present in real world equipment.

## The EDP HAN bus

The particular case that motivated the introduction of this custom integration project was the Portuguese smart meter InovGrid scenario. 
When the utility company EDP started planning the transition to an all digital smart meter network,it decided that the 
communication between the equipment and the user client, as well as other equipment in the grid, had to follow strict 
protocol guidelines that they defined in this document:

[DEF-C44-509.pdf](docs/specs/DEF-C44-509.pdf)

This specification takes the Modbus specification as the baseline, and complies with it regarding the structuring of the frames and all 
the definitions related to the communication, but extends it by allowing individual registers to have a variable length, instead of the 
fixed 16 bits of the original Modbus specification.

The EDP HAN specification does not define a specific limit to how many bytes a single register can have, but the ultimate constraint 
is the 256 bytes maximum length of the Modbus frame.

### Structure of the request / response:

Request:

| Byte 0        | Byte 1                 | Byte 2                 | Byte 3                            | Byte 4                           |
|:--------------|:-----------------------|:-----------------------|:----------------------------------|:---------------------------------|
| Function code | Starting Address (MSB) | Starting Address (LSB) | Quantity of Input Registers (MSB) |Quantity of Input Registers (LSB) |

Response:

| Byte 0        | Byte 1         | Byte 2      | Byte 3          |   Byte 4    | ...       | Byte n      |
|:--------------|:---------------|:------------|:----------------|:------------|:----------|-------------|
| Function code | Byte Count (m) | Data byte m | Data byte m - 1 | ...         | ...       | Data byte 0 |

Constraints:

```
n < 256 bytes
m <= n - 2
m % 2 == 0
```

According to the HAN specification, if m (register size), is not an even number, then the value needs to be padded with an extra byte (0x00) at the end.

## Installation

### HACS (recommended)

Have [HACS](https://hacs.xyz/) installed, this will allow you to update easily.

<a href="https://my.home-assistant.io/redirect/hacs_repository/?owner=AlexandrErohin&repository=home-assistant-tplink-router&category=integration" target="_blank"><img src="https://my.home-assistant.io/badges/hacs_repository.svg" alt="Open your Home Assistant instance and open a repository inside the Home Assistant Community Store." /></a>

or go to <b>Hacs</b> and search for `HANbus`.

### Manual

1. Locate the `custom_components` directory in your Home Assistant configuration directory. It may need to be created.
2. Copy the `custom_components/hanbus` directory into the `custom_components` directory.
3. Restart Home Assistant.

## Configuration

<!--
HANbus integration is configured via the GUI. See [the HA docs](https://www.home-assistant.io/getting-started/integration/) for more details.



1. Go to the <b>Settings</b>-><b>Devices & services</b>.
2. Click on `+ ADD INTEGRATION`, search for `HANbus`.
3. Fill the modbus device details.
4. Click `SUBMIT`

-->