# Batch Mixing Sequence Control System

## Project Overview
This repository contains a functional PLC ladder logic program designed for an automated batch process using Allen-Bradley MicroLogix architecture. The system safely manages material input, analog scaling, a timed mixing cycle, and controlled fluid discharge via automated valves.

## Engineering Logic Breakdown
The control software relies on event-driven state sequencing and strict safety interlocking:

1. **System Latching & Interlocks:** Uses standard seal-in rungs to initiate operations via `START_PB`, requiring active sensor feedback before energizing output solenoids or heating elements.
2. **Data Manipulation & Scaling:** Incorporates a Scale (`SCL`) block to process raw analog input data from plant instrumentation into engineering units.
3. **Conditional Execution:** Employs a Greater Than (`GRT`) block to dynamically monitor process thresholds. Once the scaled variable clears the setpoint, the sequence triggers an on-delay timer (`TON`) to manage the precise mixing timeline.

---

## Control Logic Architecture

### 1. Process Initiation and Heating Interlocks
This section shows the initial latching networks handling the startup cycle and element interlocks.

![System Interlocking](logic_part1.png)

### 2. Analog Scaling & Sequenced Mixing Timers
This section demonstrates the conversion of the process variable and the dynamic activation of the mixer timer loops once parameters are satisfied.

![Timer Logic](logic_part2.png)

---

## System Configuration
- **Processor Platform:** Allen-Bradley MicroLogix / 1761 Micro-Discrete
- **Development Environment:** RSLogix 500 / Virtual Workstation Deployment
- **Instruction Set Utilized:** Bit interfaces (XIC, XIO, OTE), Data Conversion (SCL), Relational Compare (GRT), Timers (TON)
---

## Technical Process Breakdown

### 1. Safety Permissives & Interlocks
* Monitored via a master latch circuit requiring an active E-Stop status, zero fault signals, and a verified closed drain valve before the cycle can initialize.
* If any safety condition is breached or the Stop button is pressed, the master `System_Active` bit drops out instantly, isolating all physical outputs.

### 2. Volumetric Charging (Analog Data Scaling)
* Utilizes analog input processing to scale raw channel integers (e.g., from a 4-20mA level transmitter) into physical volumetric engineering units (Liters).
* High/Low comparison instructions (`GEQ`, `LES`) drive precise sequential charging for Feed Valve A and Feed Valve B until the target batch volume is reached.

### 3. Agitation & Thermal Control
* Activates the mixer motor via explicit Timer On Delay (`TON`) instruction blocks upon completion of the filling phase.
* Includes hard safety interlocks requiring active mixer execution and minimum tank volume verification before enabling heating elements to prevent element burnout.

### 4. Discharge Phase & Reset
* Triggers automatically upon timer completion, opening the discharge valve to empty the vessel.
* The sequence terminates and resets the step counter back to the idle state once the scaled analog thresholds verify the vessel is fully cleared.
