# Workshop Activity Report — Day 1

**Date:** 17 November 2026  
**Department:** Electronics & Communication Engineering (ECE)  
**Topic:** Introduction to Microcontrollers, Embedded C & Arduino Programming  
**Platform:** Arduino UNO (ATmega328P) & Arduino IDE  

---

## 1. Overview & Objectives

The primary objective of Day 1 was to bridge foundational theoretical concepts of embedded systems with hands-on microcontroller programming. The session focused on:
- Understanding the architectural and functional distinctions between microprocessors and microcontrollers.
- Configuring the Arduino Integrated Development Environment (IDE) and establishing hardware communication.
- Mastering basic General Purpose Input/Output (GPIO) digital control using Embedded C/C++.
- Analyzing timing mechanisms: transitioning from simple blocking delays (`delay()`) to responsive, non-blocking timing using hardware timer registers via `millis()`.
- Implementing bidirectional Serial (UART) communication for interactive system control.

---

## 2. Tools & Hardware Specifications

### Hardware Components
| Component | Specification / Details |
| :--- | :--- |
| **Development Board** | Arduino UNO Rev3 |
| **Microcontroller (MCU)** | Microchip ATmega328P (8-bit AVR RISC architecture) |
| **Operating Voltage / Clock** | 5V DC / 16 MHz external crystal oscillator |
| **Memory** | 32 KB Flash (0.5 KB used by bootloader), 2 KB SRAM, 1 KB EEPROM |
| **I/O Interfaces** | 14 Digital I/O Pins (6 PWM capable), 6 Analog Inputs (10-bit ADC) |
| **Onboard Actuator** | User-programmable LED wired to Digital Pin 13 (`LED_BUILTIN`) |
| **Connectivity** | USB Type-A to Type-B programming cable |

### Software Environment
- **IDE:** Arduino IDE (v2.x)
- **Compiler / Toolchain:** AVR-GCC (Embedded C/C++)
- **Communication Utility:** Built-in Serial Monitor (Baud Rate: 9600 bps)

---

## 3. Theoretical Foundations

### 3.1 Microcontroller (MCU) vs. Microprocessor (MPU)
A central concept discussed was why microcontrollers are preferred for dedicated embedded systems compared to general-purpose microprocessors:

| Parameter | Microprocessor (e.g., Intel Core, ARM Cortex-A) | Microcontroller (e.g., ATmega328P, STM32) |
| :--- | :--- | :--- |
| **Integration** | CPU only; requires external RAM, ROM, and I/O chips | Single-chip IC containing CPU, RAM, Flash ROM, Timers, and GPIOs |
| **Primary Role** | High-performance, multitasking, general computing | Real-time, application-specific control tasks |
| **Power Consumption**| Higher power consumption (watts to tens of watts) | Ultra-low power consumption (milliwatts or microwatts) |
| **Cost & Footprint** | Higher circuit complexity and cost | Compact, low component count, cost-effective |
| **Operating System** | Commonly runs full OS (Linux, Windows) | Bare-metal firmware or lightweight RTOS |

### 3.2 Anatomy of an Arduino Sketch
Every standard Embedded C Arduino program contains two mandatory routines:
1. `void setup()`: Executes once at power-up or system reset. Used to configure pin modes, initialize serial communications, and load default states.
2. `void loop()`: Executes cyclically and continuously as long as the board remains powered. Handles active polling, input reading, state machines, and output actuation.

### 3.3 Timing Techniques: Blocking vs. Non-Blocking
- **Blocking (`delay(ms)`):** Suspends execution for the specified duration. During this interval, the MCU cannot process serial inputs, monitor button presses, or handle other I/O operations (except hardware interrupts).
- **Non-Blocking (`millis()`):** Reads the internal millisecond counter running since board startup. By comparing the current time with an elapsed timestamp, the MCU handles periodic tasks without freezing the main execution cycle.

---

## 4. Practical Activities & Experiments

### Activity 1: Basic LED Blinking (Blocking Delay)

#### Objective
Establish a successful toolchain workflow (code editing, compiling, and flashing) to toggle the onboard LED at a periodic 1-second interval.

#### Circuit / Pin Assignment
- **Actuator:** Onboard surface-mount LED mapped internally to **Digital Pin 13** (`LED_BUILTIN`).
- **State:** `HIGH` (5V, LED ON) / `LOW` (0V, LED OFF).

#### Source Code
```cpp
/*
 * Experiment 1: Basic LED Blink using Blocking Delay
 * Description: Alternates the onboard LED state every 1000 milliseconds.
 */

void setup() {
  // Configure Digital Pin 13 as a digital output
  pinMode(LED_BUILTIN, OUTPUT);
}

void loop() {
  digitalWrite(LED_BUILTIN, HIGH); // Drive pin HIGH (5V) -> Turn LED ON
  delay(1000);                      // Halt execution for 1000 ms (1 s)
  digitalWrite(LED_BUILTIN, LOW);  // Drive pin LOW (0V) -> Turn LED OFF
  delay(1000);                      // Halt execution for 1000 ms (1 s)
}
```

#### Working Analysis
1. `pinMode(LED_BUILTIN, OUTPUT)` sets the Data Direction Register (`DDRB`) bit for Pin 13 to output.
2. `digitalWrite()` modifies the PORT register bit to switch between logic HIGH (5V) and LOW (0V).
3. The LED blinks with a frequency of \(f = \frac{1}{T} = \frac{1}{2\text{ s}} = 0.5\text{ Hz}\) at a 50% duty cycle.

---

### Activity 2: Serial-Controlled Multimode LED with Non-Blocking Timing

#### Objective
Design an interactive command system via the UART Serial Monitor allowing the user to select LED behavior dynamically without blocking MCU execution.

#### Functional Modes
- `'1'`: Steady ON
- `'0'`: Steady OFF
- `'b'`: Asynchronous Blink at 500 ms intervals (Non-blocking timing via `millis()`)

#### Source Code
```cpp
/*
 * Experiment 2: Serial-Controlled Multimode LED with Non-Blocking Delay
 * Description: Interprets user keystrokes over Serial (UART) to switch modes:
 *              '1' -> LED ON
 *              '0' -> LED OFF
 *              'b' -> Non-blocking blink (500 ms toggle interval)
 */

char mode = '0';                     // Operating mode register: '0' = OFF, '1' = ON, 'b' = BLINK
unsigned long previousMillis = 0;   // Stores the timestamp of the last LED state toggle
const unsigned long interval = 500; // Toggle interval in milliseconds (blink rate)
bool ledState = LOW;                // Current LED state tracker

void setup() {
  pinMode(LED_BUILTIN, OUTPUT);     // Initialize pin 13 as digital output
  Serial.begin(9600);               // Initialize UART communication at 9600 baud rate
  Serial.println("========================================");
  Serial.println("Arduino Serial LED Controller Initialized");
  Serial.println("Commands: [1] = ON | [0] = OFF | [b] = BLINK");
  Serial.println("========================================");
}

void loop() {
  // 1. Asynchronously poll and read incoming Serial commands
  if (Serial.available() > 0) {
    char c = Serial.read();
    if (c == '0' || c == '1' || c == 'b') {
      mode = c;
      Serial.print("Active Mode Updated -> ");
      Serial.println(mode);
    }
  }

  // 2. Execute selected operational mode
  if (mode == '1') {
    digitalWrite(LED_BUILTIN, HIGH);
  } else if (mode == '0') {
    digitalWrite(LED_BUILTIN, LOW);
  } else if (mode == 'b') {
    // Non-blocking timer routine
    unsigned long currentMillis = millis();
    if (currentMillis - previousMillis >= interval) {
      previousMillis = currentMillis;  // Update checkpoint timestamp
      ledState = !ledState;            // Invert logic state
      digitalWrite(LED_BUILTIN, ledState);
    }
  }
}
```

#### Working Analysis
1. `Serial.available()` returns the number of unread bytes in the 64-byte hardware serial ring buffer.
2. `Serial.read()` consumes individual ASCII characters sent from the host PC.
3. The non-blocking timing loop computes \(\Delta t = \text{currentMillis} - \text{previousMillis}\). When \(\Delta t \ge 500\text{ ms}\), the LED state toggles while remaining immediately responsive to any incoming keystroke.

---

## 5. Summary of Key Learnings & Competencies

- **Development Workflow:** Acquired end-to-end proficiency in opening, writing, verifying (compiling), selecting appropriate board (`Arduino Uno`) and COM port, and flashing firmware.
- **Embedded C Syntax:** Gained working knowledge of standard types (`char`, `bool`, `unsigned long`), state persistence, conditional logic, and modular functions.
- **Hardware Abstraction Layer (HAL):** Learned how Arduino core functions (`pinMode`, `digitalWrite`, `millis`) map to low-level hardware registers.
- **Asynchronous Design Thinking:** Understood the limitations of `delay()` in real-time embedded systems and mastered timestamp-based scheduling using `millis()`.
- **Serial Debugging & Interfacing:** Implemented serial communication protocols at 9600 baud for telemetry, runtime monitoring, and external command handling.

---

## 6. Troubleshooting & Best Practices Encountered

> [!NOTE]
> **Common Issues & Resolutions:**
> - **COM Port Not Detected:** Check device manager for missing USB-to-UART bridge drivers (e.g., CH340 or Atmega16U2) and ensure the USB cable supports data transfer (not charge-only).
> - **Garbage Characters in Serial Monitor:** Ensure the baud rate selected in the Serial Monitor matches the firmware configuration (`9600 baud`).
> - **Variable Overflow:** Always declare timing variables associated with `millis()` as `unsigned long` (32-bit unsigned integer, safe up to ~49.7 days without overflow).

---

## 7. Next Steps & Planned Activities

- [ ] Interfacing external discrete LEDs and computing current-limiting resistor values using Ohm's Law:
  $$R = \frac{V_{CC} - V_{F}}{I_{F}}$$
- [ ] Experimenting with Pulse Width Modulation (PWM) via `analogWrite()` for variable brightness control.
- [ ] Interfacing digital inputs (push-buttons) with internal pull-up resistors (`INPUT_PULLUP`) and software switch debouncing.
- [ ] Exploring hardware interrupt service routines (`attachInterrupt()`) for time-critical event handling.
