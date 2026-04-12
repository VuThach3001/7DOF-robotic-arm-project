# Requirements on 7DOF Robotic Arm - System Architecture

| Attribute           | Value              |
| ------------------- | ------------------ |
| **Document ID**     | 7DOF_RS_001        |
| **Document Status** | INDEVELOP          |
| **Version**         | 0.1.0              |
| **Date**            | 2026-04-12         |
| **Author**          | Thach Nguyen Ba Vu |

---

## Document Change History

| Version | Date       | Author             | Change Description |
| ------- | ---------- | ------------------ | ------------------ |
| 0.1.0   | 2026-04-12 | Thach Nguyen Ba Vu | Initial draft      |

---

## Table of Contents

- [Requirements on 7DOF Robotic Arm - System Architecture](#requirements-on-7dof-robotic-arm---system-architecture)
  - [Document Change History](#document-change-history)
  - [Table of Contents](#table-of-contents)
  - [1. Scope](#1-scope)
  - [2. References](#2-references)
  - [3. Constraints and Assumptions](#3-constraints-and-assumptions)
  - [4. Requirements](#4-requirements)
    - [4.1 Requirement Category](#41-requirement-category)
      - [\[RS\_SA\_00001\] ESP32S3 Requirement](#rs_sa_00001-esp32s3-requirement)
      - [\[RS\_SA\_00002\] STM32F411CEU6 Requirement](#rs_sa_00002-stm32f411ceu6-requirement)
  - [5. References](#5-references)

---

## 1. Scope

Describe the scope of this requirements document.

---

## 2. References

| #   | Document Title  | Version |
| --- | --------------- | ------- |
| [1] | Reference Title | 1.0.0   |

---

## 3. Constraints and Assumptions

Describe constraints and assumptions.

---

## 4. Requirements

### 4.1 Requirement Category

#### [RS_SA_00001] ESP32S3 Requirement

| Attribute    | Value      |
| ------------ | ---------- |
| **Type**     | Functional |
| **Status**   | Indevelop  |
| **Priority** | High       |

**Description:**  
- The ESP32 acts as the high-level controller responsible for computation-heavy, non-real-time tasks and system coordination.
- It should:
  - Run high-level algorithms (IK, trajectory planning)
  - Manage communication with external systems (PC, UI, cloud (idea))
  - Act as a bridge between user commands and low-level control
  - Optionally handle vision (basic camera processing if used)

**Rationale:**  
- The ESP32 is chosen because:
  - It has higher processing capability than STM32 (especially for floating-point math)
  - Built-in Wi-Fi / Bluetooth enables remote control and monitoring
  - Supports multitasking (FreeRTOS)

- This prevents:
  - Timing issues
  - Control instability
  - Overloading the real-time controller (STM32)

**Use Case:**
**A. Inverse Kinematics Computation**
```
Input: End-effector position (x, y, z)
Output: Joint angles (θ1 → θ7)
```
- Compute IK for 7DOF arm
- Handle redundancy (multiple solutions)

**B. Trajectory Planning**
- Generate smooth motion:
```
Point A → Point B with velocity constraints
```
- Interpolate joint angles over time

**C. Communication Hub**

- Receive commands from:
  - PC / UI
  - Mobile app (via Wi-Fi)
- Send commands to STM32 via:
  - UART / CAN / SPI

**D. System Coordination**
- Manage states machine (IDLE, STANDBY,...)
- Synchronize multiple modules

**E. Optional Vision Processing**
- Camera input.
- Object detection / tracking (lightweight).

**Dependencies:**
- TBD

#### [RS_SA_00002] STM32F411CEU6 Requirement

**Description:**  
- The STM32 acts as the real-time motor controller, directly controlling actuators and processing sensor feedback.
- It should:
  - Execute motor control loops (PID)
  - Read sensors (encoders, limit switches)
  - Ensure deterministic timing (real-time behavior)
  - Enforce safety constraints

**Rationale:**  
- The STM32 is chosen because:

- Provides real-time performance (deterministic timing)
- Has hardware peripherals:
  - Timers (PWM generation)
  - Encoder interfaces
  - ADC (sensor reading)
- More reliable for low-level control loops

**Use Cases:**
**A. Motor Control (Core Function)**
```
Input: Target joint angle
Output: PWM signal to motor driver
```
- Run PID loop for each joint (7 DOF)
- Maintain position / velocity

**B. Sensor Feedback Processing**
- Read:
  - Encoders (joint position)
  - Limit switches (safety)
  - Current sensors (optional)
- Filter noise and provide stable data

**C. Real-Time Execution**
- Control loop frequency:
```
1 kHz (typical)
```
- Ensure:
  - No jitter
  - Deterministic response

**D. Safety Handling**
- Emergency stop
- Joint limits enforcement
- Fault detection:
  - Overcurrent
  - Encoder failure

## 5. References

---

