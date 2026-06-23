# Portable Tilt Status Indicator using STM32F446RE and MPU6050

## 1. Overview

This assignment asks participants to develop a **Portable Tilt Status Indicator** using:

- **STM32F446RE**
- **MPU6050**
- **GPIO**
- **USART2**
- **I2C1**

The system reads accelerometer data from the MPU6050 over I2C, sends sensor values and status over UART, and indicates the detected condition using LEDs.

This is a short, hands-on embedded systems exercise suitable for a **90-minute FDP lab session**.

---

## 2. Problem Scenario

A company is developing a **portable tilt-status indicator** for small electronic equipment that must remain approximately level during operation and handling.

The system uses an **STM32F446RE** microcontroller and an **MPU6050 inertial sensor**. Its purpose is to continuously monitor device orientation and determine whether the equipment is:

- in a **NORMAL** position, or
- in a **TILT** condition beyond an allowed threshold

To support development and diagnostics:

- the **MPU6050 shall be interfaced using I2C**
- sensor values and system status shall be sent over **UART**
- **GPIO LEDs** shall indicate the current state

The complete implementation must be written in **`main.c` only**.

---

## 3. Assignment Objective

Develop firmware that:

1. Initializes **GPIO**, **USART2**, and **I2C1**
2. Detects and initializes the **MPU6050**
3. Reads **accelerometer X, Y, Z** values
4. Sends sensor values and state information over **UART**
5. Detects whether the board is:
   - **NORMAL**
   - **TILT**
6. Updates LEDs based on the detected state

---

## 4. Learning Outcomes

By completing this assignment, participants should be able to:

- configure and use **GPIO** for LED control
- configure and use **UART** for serial debugging
- interface **MPU6050 over I2C**
- perform **register write and register read**
- acquire raw accelerometer data
- implement simple **threshold-based decision logic**
- integrate sensor reading, UART logging, and GPIO indication in one application

---

## 5. Functional Requirements

### 5.1 Hardware Interfaces

The application shall use:

- **GPIO**
  - for LED indication
- **USART2**
  - for serial terminal output
- **I2C1**
  - for communication with MPU6050

---

### 5.2 System States

The system shall support two states only.

#### State 1: NORMAL
- Board is approximately level
- **Green LED ON**

#### State 2: TILT
- Board tilt exceeds the threshold
- **Red LED ON**

> Only one LED shall remain ON at a time.

---

### 5.3 Startup Behavior

On power-up, the firmware shall:

1. initialize **HAL**
2. initialize system clock
3. initialize **GPIO**
4. initialize **USART2**
5. initialize **I2C1**
6. print startup messages over UART
7. wake up and configure the MPU6050
8. verify sensor presence using `WHO_AM_I`

If the sensor is not detected, the firmware shall print an error message.

---

### 5.4 Runtime Behavior

During execution, the firmware shall:

1. read accelerometer values:
   - `Ax`
   - `Ay`
   - `Az`
2. print the values over UART
3. determine whether the board is:
   - `NORMAL`
   - `TILT`
4. update LEDs
5. repeat every **300 ms**

---

## 6. Simplified Detection Logic

To keep the assignment suitable for a short FDP session, **do not use trigonometric calculations** such as pitch/roll using `atan()`.

Use simple raw-value threshold logic.

### Decision Rule

```c
if (abs(Ax) > 6000 || abs(Ay) > 6000)
    state = TILT;
else
    state = NORMAL;
````

### Interpretation

* If the absolute value of **Ax** or **Ay** exceeds `6000`, the board is treated as **TILT**
* Otherwise, it is treated as **NORMAL**

***

## 7. Exact Peripheral Configuration

### 7.1 USART2 Configuration

Use **USART2** because it is connected to the ST-Link Virtual COM port on the Nucleo board.

#### Settings

* **Peripheral:** `USART2`
* **Baud rate:** `115200`
* **Word length:** `8 Bits`
* **Stop bits:** `1`
* **Parity:** `None`
* **Mode:** `TX/RX`
* **Hardware flow control:** `None`
* **Oversampling:** `16`

#### Pins

* `PA2` → `USART2_TX`
* `PA3` → `USART2_RX`

***

### 7.2 I2C1 Configuration

Use **I2C1** for MPU6050 communication.

#### Settings

* **Peripheral:** `I2C1`
* **Clock speed:** `100 kHz`
* **Addressing mode:** `7-bit`
* **Dual address mode:** `Disable`
* **General call mode:** `Disable`
* **No stretch mode:** `Disable`

#### Pins

* `PB8` → `I2C1_SCL`
* `PB9` → `I2C1_SDA`

***

### 7.3 GPIO Configuration

Use LEDs as follows:

* `PA5` → **Green LED** → `NORMAL`
* `PB7` → **Red LED** → `TILT`

#### GPIO Settings

* **Mode:** Output Push-Pull
* **Pull:** No Pull
* **Speed:** Low

***

## 8. MPU6050 Configuration Details

### 8.1 Device Address

* **MPU6050 I2C address:** `0x68`
* **HAL shifted address:** `0xD0`

***

### 8.2 Registers to Configure

#### 1. Wake-up Register

* **Register:** `PWR_MGMT_1`
* **Address:** `0x6B`
* **Value:** `0x00`

Purpose: Wake MPU6050 from sleep mode

#### 2. Accelerometer Configuration

* **Register:** `ACCEL_CONFIG`
* **Address:** `0x1C`
* **Value:** `0x00`

Meaning: Accelerometer full scale = **±2g**

#### 3. Gyroscope Configuration

* **Register:** `GYRO_CONFIG`
* **Address:** `0x1B`
* **Value:** `0x00`

Meaning: Gyroscope full scale = **±250 dps**

> Gyroscope configuration may still be done for completeness, although gyroscope values are not used in the final decision logic.

#### 4. Device Identification

* **Register:** `WHO_AM_I`
* **Address:** `0x75`
* **Expected Value:** `0x68`

***

## 9. Registers to Read

For this assignment, only accelerometer data is required.

### Accelerometer Registers

* `ACCEL_XOUT_H` → `0x3B`
* `ACCEL_XOUT_L` → `0x3C`
* `ACCEL_YOUT_H` → `0x3D`
* `ACCEL_YOUT_L` → `0x3E`
* `ACCEL_ZOUT_H` → `0x3F`
* `ACCEL_ZOUT_L` → `0x40`

### Recommended Read Method

Read **6 bytes starting from `0x3B`**

Then combine them into signed 16-bit values:

* `Ax`
* `Ay`
* `Az`

***

## 10. Tasks to Implement

### Task 1: Peripheral Initialization

Configure and initialize:

* GPIO
* USART2
* I2C1

#### Expected UART Output

```text
System Boot
GPIO OK
UART OK
I2C OK
```

***

### Task 2: MPU6050 Detection and Initialization

Implement:

* write `0x00` to `PWR_MGMT_1` (`0x6B`)
* read `WHO_AM_I` (`0x75`)
* verify returned value is `0x68`

#### Expected UART Output

If detected:

```text
MPU6050 Detected
```

If not detected:

```text
MPU6050 Not Detected
```

***

### Task 3: Read Accelerometer Data

Read:

* `Ax`
* `Ay`
* `Az`

#### Expected UART Output

```text
AX=120 AY=-80 AZ=16320
```

***

### Task 4: Implement Tilt Detection

Use the logic:

```c
if (abs(Ax) > 6000 || abs(Ay) > 6000)
    state = TILT;
else
    state = NORMAL;
```

#### Expected UART Output

```text
STATE = NORMAL
```

or

```text
STATE = TILT
```

***

### Task 5: LED Indication

Implement LED control as follows:

* `NORMAL` → Green LED ON
* `TILT` → Red LED ON

Only one LED should be ON at a time.

***

## 11. Suggested Program Flow in `main.c`

Since the full implementation must be written in **`main.c` only**, the following flow is recommended.

### Suggested Helper Functions

```c
uint8_t MPU6050_Check(void);
void MPU6050_Init(void);
void MPU6050_Read_Accel(void);
void UART_Print(char *msg);
void Update_LED(uint8_t state);
```

### Recommended Main Flow

1. `HAL_Init()`
2. `SystemClock_Config()`
3. `MX_GPIO_Init()`
4. `MX_USART2_UART_Init()`
5. `MX_I2C1_Init()`
6. print startup messages
7. check MPU6050 presence
8. initialize MPU6050
9. inside `while(1)`:
   * read `Ax`, `Ay`, `Az`
   * apply tilt logic
   * update LEDs
   * print values and state
   * delay `300 ms`

***

## 12. Fixed Values for Participants

### UART

```text
USART2
115200 baud
8 data bits
1 stop bit
No parity
TX/RX mode
```

### I2C

```text
I2C1
100 kHz
7-bit addressing
```

### MPU6050

```text
Device address = 0x68
HAL shifted address = 0xD0
WHO_AM_I register = 0x75
Expected WHO_AM_I value = 0x68
PWR_MGMT_1 register = 0x6B
Wake-up value = 0x00
ACCEL_CONFIG register = 0x1C
ACCEL_CONFIG value = 0x00
GYRO_CONFIG register = 0x1B
GYRO_CONFIG value = 0x00
```

### Accelerometer Read Start Address

```text
0x3B
```

### Read Period

```text
300 ms
```

### Threshold Rule

```text
TILT if |Ax| > 6000 OR |Ay| > 6000
Else NORMAL
```

***

## 13. Test Cases

### TC1: Power-On Boot Test

#### Objective

Verify startup and peripheral initialization.

#### Procedure

* power the board
* open the serial terminal

#### Expected Result

```text
System Boot
GPIO OK
UART OK
I2C OK
```

***

### TC2: Sensor Detection Test

#### Objective

Verify MPU6050 communication over I2C.

#### Procedure

* connect MPU6050 correctly
* run the program

#### Expected Result

```text
MPU6050 Detected
```

#### Negative Variation

Disconnect the sensor or wiring.

#### Expected Result

```text
MPU6050 Not Detected
```

***

### TC3: Raw Accelerometer Data Test

#### Objective

Verify accelerometer data acquisition.

#### Procedure

* keep the board steady
* observe UART output
* slowly change board orientation

#### Expected Result

* `Ax`, `Ay`, `Az` update periodically
* values change when orientation changes

Example:

```text
AX=100 AY=-120 AZ=16280
```

***

### TC4: NORMAL State Test

#### Objective

Verify normal state detection.

#### Procedure

* keep the board approximately flat on the table

#### Expected Result

* Green LED ON
* UART prints:

```text
STATE = NORMAL
```

***

### TC5: TILT State Test

#### Objective

Verify tilt detection.

#### Procedure

* tilt the board left, right, forward, or backward

#### Expected Result

* Red LED ON
* UART prints:

```text
STATE = TILT
```

***

### TC6: Return-to-Normal Test

#### Objective

Verify recovery from tilted state.

#### Procedure

* tilt the board until `TILT` is detected
* place it flat again

#### Expected Result

* Red LED turns OFF
* Green LED turns ON
* UART prints:

```text
STATE = NORMAL
```

***
