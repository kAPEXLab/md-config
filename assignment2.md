## Assignment 2

# **Smart Temperature Monitoring & Alert System**

***

## 📖 **Introduction (Real-World Context)**

In automotive ECUs (e.g., battery thermal management systems), continuous monitoring of temperature is critical. Multiple software components interact:

* Sensor acquisition
* Data processing
* Communication/logging

Such systems must ensure **data integrity**, **task prioritization**, and **safe resource sharing**, especially when multiple tasks access shared peripherals like UART.

This assignment simulates a **real-time temperature monitoring ECU** using:

* LM35 temperature sensor (ADC)
* FreeRTOS (CMSIS V2 APIs)
* Task communication via queues
* Mutual exclusion using mutex

***

## 🎯 **Objective**

Design a real-time embedded application that:

* Reads temperature from LM35 (ADC)
* Processes and classifies temperature
* Logs system status via UART (protected by mutex)
* Uses FreeRTOS constructs (tasks, queue, mutex)

***

## ⚙️ **System Requirements**

### ✅ Hardware

* STM32F446RE (NUCLEO board)
* LM35 Temperature Sensor connected to ADC channel
* UART (USB/Virtual COM)

### ✅ Software

* STM32CubeIDE
* FreeRTOS (CMSIS V2)

***

## 🏗️ **System Architecture**

```
        +-------------------+
        |   ADC Task        |
        | (Sensor Read)     |
        +--------+----------+
                 |
                 | Queue (Temperature Data)
                 v
        +-------------------+
        | Processing Task   |
        | (Decision Logic)  |
        +--------+----------+
                 |
                 | Queue (Status Message)
                 v
        +-------------------+
        | UART Task         |
        | (Logging Output)  |
        +-------------------+

          [Mutex protects UART]
```

***

## 🔧 **Implementation Requirements**

### ✅ 1. FreeRTOS Objects

* 3 Tasks
* 1 Queue (ADC → Processing)
* 1 Queue (Processing → UART)
* 1 Mutex (for UART access)

***

## 🧩 **Tasks to Implement**

### 🔹 **Task 1: ADC Task**

* Periodicity: 500 ms
* Read ADC value from LM35
* Convert to temperature:
  ```
  Temp (°C) = (ADC_Value * 3.3 / 4095) * 100
  ```
* Send temperature to queue

***

### 🔹 **Task 2: Processing Task**

* Wait for data from ADC queue
* Classify temperature:

| Temperature Range | Status   |
| ----------------- | -------- |
| < 25°C            | NORMAL   |
| 25–40°C           | WARNING  |
| > 40°C            | CRITICAL |

* Send status message to UART queue

***

### 🔹 **Task 3: UART Task**

* Wait for message from processing queue
* Take **mutex**
* Print formatted message:
  ```
  Temp: XX.X °C | Status: XXXXX
  ```
* Release mutex

***

## 🔐 **Mutex Usage Requirement**

* UART must be a **shared resource**
* Even if only one task currently uses UART, enforce mutex usage (industry practice)

***

## 🧪 **Test Cases**

### ✅ **Test Case 1: Normal Condition**

* Input: \~20°C
* Expected Output:
  ```
  Temp: 20.X °C | Status: NORMAL
  ```

***

### ✅ **Test Case 2: Warning Condition**

* Input: \~30°C
* Expected Output:
  ```
  Temp: 30.X °C | Status: WARNING
  ```

***

### ✅ **Test Case 3: Critical Condition**

* Input: \~45°C (heat sensor manually)
* Expected Output:
  ```
  Temp: 45.X °C | Status: CRITICAL
  ```

***

### ✅ **Test Case 4: Queue Communication**

* Verify:
  * No data loss
  * Continuous data flow between tasks

***

### ✅ **Test Case 5: Mutex Behavior**

* Add **temporary debug print in another task**
* Ensure:
  * Output is not corrupted
  * No overlapping prints

***
