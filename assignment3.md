# Problem Statement

## Title

**CAN-Based Temperature Monitoring and Indication System Using STM32 Nucleo Boards**

---

## Background

Modern automotive Electronic Control Units (ECUs) continuously exchange sensor data over the Controller Area Network (CAN). Temperature information from sensors is typically acquired by one ECU and transmitted to other ECUs for monitoring, warning indication, and control actions.

In this project, students will develop a distributed embedded system consisting of two STM32 Nucleo boards communicating over a CAN network.

---

## Objective

Design and implement a two-node embedded system using STM32 Nucleo boards, where:

- One node acquires temperature from an LM35 sensor.
- The measured temperature is transmitted periodically over CAN.
- A second node receives the temperature information and indicates the temperature range using coloured LEDs.
- A switch connected to the first node sends a CAN command that turns OFF all LEDs on the second node.

---

## System Architecture

```text
                    CAN Bus
+-------------------------------------+
|                                     |
|                                     |
v                                     v

+-------------------+      +-------------------+
| STM32 Nucleo      |      | STM32 Nucleo      |
| Node 1            |      | Node 2            |
|                   |      |                   |
| LM35 Sensor       |----->| Temperature       |
|                   | CAN  | Indication LEDs   |
| Push Button       |----->| Remote LED Reset  |
+-------------------+      +-------------------+
```

---

## Hardware Requirements

### Node 1 – Sensor ECU

- STM32 Nucleo Board
- LM35 Temperature Sensor
- Push Button Switch
- CAN Transceiver

### Node 2 – Indicator ECU

- STM32 Nucleo Board
- CAN Transceiver
- Green LED
- Yellow LED
- Red LED

---

## Functional Requirements

### FR1. Temperature Measurement

The Sensor ECU shall:

- Acquire the LM35 output using the ADC peripheral.
- Convert the ADC reading into temperature in degree Celsius.
- Update the temperature value periodically.

---

### FR2. Temperature Transmission

The Sensor ECU shall:

- Transmit the temperature value over the CAN bus every 1 second.
- Use a predefined CAN Identifier for temperature messages.

---

### FR3. Temperature Indication

The Indicator ECU shall:

- Receive temperature messages from the CAN bus.
- Illuminate LEDs according to the temperature range.

#### Temperature Indication Logic

| Temperature Range | LED Status |
|------------------|------------|
| Below 20°C | Green LED ON |
| 20°C to 35°C | Yellow LED ON |
| Above 35°C | Red LED ON |

**Note:** Only one LED shall be ON at any given time.

---

### FR4. Remote LED OFF Command

A push button shall be connected to the Sensor ECU.

When the switch is pressed:

- A CAN command shall be transmitted.
- The Indicator ECU shall immediately turn OFF all LEDs.

---

### FR5. Recovery to Normal Operation

After receiving the LED OFF command:

- All LEDs shall remain OFF.
- Upon reception of the next valid temperature message, normal temperature indication shall resume.

---

## CAN Message Design

### Temperature Message

```text
CAN ID : 0x100
DLC    : 1

Byte0  : Temperature (°C)
```

#### Example

```text
Temperature = 25°C

CAN ID : 0x100
DATA   : 0x19
```

---

### LED OFF Command Message

```text
CAN ID : 0x101
DLC    : 1

Byte0  : 0xAA
```

---

## Software Requirements

### Node 1 (Sensor ECU)

#### ADC Module

- ADC initialisation
- Temperature acquisition
- ADC to temperature conversion

#### GPIO Module

- Push button detection
- Debounce handling

#### CAN Module

- CAN initialisation
- CAN transmission

#### Application Layer

- Read temperature
- Generate CAN temperature message
- Detect switch press
- Generate LED OFF command

---

### Node 2 (Indicator ECU)

#### GPIO Module

- LED control functions

#### CAN Module

- CAN initialisation
- CAN receive interrupt handling

#### Application Layer

- Process received temperature messages
- Determine temperature range
- Control LEDs
- Process LED OFF command

---

## Expected Behaviour

### Scenario 1: Low Temperature

```text
Temperature = 15°C

Green LED  : ON
Yellow LED : OFF
Red LED    : OFF
```

### Scenario 2: Medium Temperature

```text
Temperature = 30°C

Green LED  : OFF
Yellow LED : ON
Red LED    : OFF
```

### Scenario 3: High Temperature

```text
Temperature = 42°C

Green LED  : OFF
Yellow LED : OFF
Red LED    : ON
```

### Scenario 4: Switch Pressed

```text
Switch Pressed on Node 1

Result:
Green LED  : OFF
Yellow LED : OFF
Red LED    : OFF
```

---

## Learning Outcomes

Upon successful completion of the project, students will be able to:

- Interface analogue sensors using STM32 ADC.
- Implement CAN communication using STM32 peripherals.
- Design CAN messages and identifiers.
- Develop multi-node embedded applications.
- Handle interrupts and event-driven software.
- Implement automotive-style ECU communication.
- Debug embedded systems using STM32CubeIDE.

---

## Bonus Challenges

### Challenge 1

Transmit temperature with 0.1°C resolution.

Example:

```text
32.5°C

Transmit:
325
```

---

### Challenge 2

Implement CAN communication timeout detection.

Requirement:

- If no temperature message is received for 5 seconds,
- Blink the Red LED at 1 Hz.

---

### Challenge 3

Detect sensor disconnection.

Requirement:

- Turn ON an additional Blue LED when the sensor value is invalid.

---

### Challenge 4

Display the received temperature on a UART terminal.

Example:

```text
Received Temperature = 28°C
```

---

## Acceptance Criteria

The system shall be considered successfully implemented when:

- Temperature is correctly acquired from the LM35 sensor.
- Temperature messages are periodically transmitted over CAN.
- LEDs correctly indicate the received temperature range.
- Switch press on Node 1 turns OFF all LEDs on Node 2.
- Communication between both STM32 Nucleo boards is reliable.
- The application demonstrates a distributed automotive ECU architecture.
