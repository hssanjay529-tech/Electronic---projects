# Workshop Activity Report — Day 2

**Date:** 18 November 2026  
**Department:** Electronics & Communication Engineering (ECE)  
**Topic:** IoT Fundamentals, ESP32 Microcontroller, Sensor Interfacing (IR & Ultrasonic), H-Bridge Motor Control & Embedded Web Servers  
**Platform:** ESP32 NodeMCU / DevKit (ESP-WROOM-32) & Arduino IDE  

---

## 1. Overview & Objectives

Building upon the fundamental microcontroller concepts from Day 1, Day 2 expanded into Internet of Things (IoT) architectures, sensor integration, actuator interfacing, and embedded web server development using the dual-core **ESP32** microcontroller.

Key objectives included:
- Configuring the ESP32 development environment in the Arduino IDE and establishing wireless connectivity over Wi-Fi (Station Mode) via a mobile hotspot.
- Interfacing digital and analog sensors: Infrared (IR) obstacle detection and HC-SR04 ultrasonic time-of-flight distance measurement.
- Implementing safe 5V-to-3.3V logic level shifting using a resistive voltage divider for sensor inputs.
- Driving and modulating DC motors using the L298N dual H-Bridge driver with hardware Pulse Width Modulation (PWM) via the ESP32 `LEDC` peripheral.
- Hosting full-stack embedded web servers (`WebServer.h`) directly on the ESP32 to serve responsive HTML5/CSS3/JavaScript web applications for bidirectional IoT control and real-time telemetry.

---

## 2. Tools & Hardware Specifications

### Hardware Components
| Component | Function / Technical Specifications |
| :--- | :--- |
| **Microcontroller** | ESP32-WROOM-32 (Tensilica Xtensa Dual-Core 32-bit LX6 @ 240 MHz, 520 KB SRAM, 4 MB Flash) |
| **Wireless Module** | Integrated 2.4 GHz Wi-Fi (802.11 b/g/n) & Bluetooth v4.2 BR/EDR + BLE |
| **IR Obstacle Sensor** | Active-LOW digital sensor module (IR emitter, photodiode receiver, onboard LM393 comparator & potentiometer) |
| **Ultrasonic Sensor** | HC-SR04 (40 kHz acoustic transceiver, 2 cm – 400 cm range, 5V operating voltage) |
| **Motor Driver** | L298N Dual H-Bridge Module (up to 2A per channel, external 12V/9V motor power, onboard 5V regulator) |
| **Actuators & Alerts** | 5V DC Toy Gear Motor, Active Buzzer (5V), 5mm Red & Green LEDs |
| **Passive Components** | $220\ \Omega$ current-limiting resistors, $1\text{ k}\Omega$ & $2\text{ k}\Omega$ resistors (voltage divider for Echo pin) |
| **Power Supply** | Micro-USB (for ESP32 logic) + External Battery Pack/DC adapter (for L298N motor rail) |

### Software Environment
- **IDE:** Arduino IDE (v2.x) with ESP32 Board Core (Espressif Systems)
- **Libraries Used:** `<WiFi.h>`, `<WebServer.h>`, `<pgmspace.h>`
- **Client Interface:** Modern Web Browser (Chrome / Edge / Safari) on smartphone or PC connected to the same local Wi-Fi network

---

## 3. Theoretical Foundations

### 3.1 ESP32 vs. Arduino UNO (ATmega328P)
| Feature | Arduino UNO (Day 1) | ESP32 DevKit (Day 2) |
| :--- | :--- | :--- |
| **Processor Architecture** | 8-bit AVR RISC @ 16 MHz | 32-bit Dual-Core Xtensa LX6 @ 240 MHz |
| **RAM / Flash** | 2 KB SRAM / 32 KB Flash | 520 KB SRAM / 4 MB SPI Flash |
| **Operating Logic Voltage** | 5.0 V logic | 3.3 V logic (requires voltage shifting for 5V signals) |
| **Connectivity** | None (USB-Serial only) | Native 2.4 GHz Wi-Fi & Bluetooth Classic/BLE |
| **PWM Generation** | Timer-based fixed frequency | Hardware LEDC peripheral (up to 16 channels, custom frequency & resolution) |

---

### 3.2 Sensor Operating Principles

#### A. Infrared (IR) Obstacle Detection Module
- **Mechanism:** Emits continuous infrared light via an IR LED. When an obstacle enters the beam path, the light reflects back into the photodiode.
- **Signal Conditioning:** An onboard LM393 comparator compares the reflected photodiode voltage against a reference set by a trim potentiometer.
- **Output Logic:** Active-LOW signal:
  $$\text{Obstacle Detected} \implies \text{Output} = \text{LOW (0V)}$$
  $$\text{Clear Path} \implies \text{Output} = \text{HIGH (3.3V/5V)}$$

#### B. HC-SR04 Ultrasonic Distance Sensor
- **Mechanism:** Emits an eight-cycle sonic burst at $40\text{ kHz}$ from the transmitter (Trig) and measures the duration until the reflected echo pulse returns to the receiver (Echo).
- **Time-of-Flight Physics:**
  $$\text{Speed of sound in dry air at } 20^\circ\text{C} \approx 343\text{ m/s} = 0.0343\text{ cm/\mu s}$$
  $$\text{Distance (cm)} = \frac{\text{Echo Pulse Width (\mu s)} \times 0.0343\text{ cm/\mu s}}{2}$$
- **Protection via Voltage Divider:**
  The HC-SR04 is powered by 5V and outputs a 5V logic pulse on its `ECHO` pin. Because ESP32 GPIOs are **3.3V-tolerant only**, a resistive divider is mandatory:
  $$V_{out} = V_{in} \times \frac{R_2}{R_1 + R_2} = 5\text{V} \times \frac{2\text{ k}\Omega}{1\text{ k}\Omega + 2\text{ k}\Omega} = 3.33\text{ V}$$

---

### 3.3 H-Bridge Operation & PWM Speed Regulation (L298N)
A standard microcontroller cannot source the high current (~500 mA to 1.5 A) demanded by DC motors. An **H-Bridge** circuit allows bidirectional current flow through four switching transistors:

| IN1 | IN2 | ENA (PWM) | Motor State |
| :---: | :---: | :---: | :---: |
| HIGH | LOW | $>0$ | Forward Rotation |
| LOW | HIGH | $>0$ | Reverse Rotation |
| LOW | LOW | Any | Motor Coast / Stop |
| HIGH | HIGH | Any | Active Dynamic Brake |

The speed is modulated by applying Pulse Width Modulation (PWM) to the `ENA` pin, varying the average voltage delivered to the armature coil:
$$V_{avg} = V_{rail} \times \left(\frac{\text{Duty Cycle}}{255}\right)$$

---

### 3.4 Embedded IoT Web Server Architecture
The ESP32 runs an HTTP server on TCP port 80. Rather than reloading the entire webpage upon every command, modern IoT design utilizes **AJAX / asynchronous `fetch()` polling**:
1. Host browser loads the static HTML/CSS UI stored in microcontroller Flash (`PROGMEM`).
2. A client-side JavaScript loop queries `/status` every 300 ms.
3. The ESP32 replies with a compact JSON payload: `{"object": true, "led": false}`.
4. The client DOM updates dynamically without full-page refreshes.

---

## 4. Practical Activities & Experiments

### Activity 1: ESP32 Wi-Fi Setup & Web Server for Remote LED Control

#### Objective
Configure the ESP32 in Station Mode (STA) to join a local Wi-Fi hotspot and host an HTTP web server with interactive endpoints to toggle an LED remotely from a browser.

#### Circuit & Pin Connections
- **Actuator:** Onboard LED connected to **GPIO 2** (`LED_PIN`).
- **Power:** ESP32 powered via USB; client connected to the assigned IP address.

#### Source Code
```cpp
/*
 * Experiment 1: ESP32 Basic Web Server for Remote LED Control
 * Description: Connects to a Wi-Fi Access Point and serves an interactive 
 *              web interface to toggle GPIO 2 ON and OFF via HTTP requests.
 */

#include <WiFi.h>
#include <WebServer.h>

// Wi-Fi Access Point Credentials
const char* ssid     = "YOUR_WIFI_HOTSPOT";
const char* password = "YOUR_WIFI_PASSWORD";

// Initialize HTTP WebServer on default TCP Port 80
WebServer server(80);

#define LED_PIN 2

void setup() {
  Serial.begin(115200);
  pinMode(LED_PIN, OUTPUT);
  digitalWrite(LED_PIN, LOW);

  // Connect to Wi-Fi network
  Serial.print("Connecting to Wi-Fi: ");
  Serial.println(ssid);
  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("\nWiFi Connected successfully!");
  Serial.print("ESP32 Local IP Address: ");
  Serial.println(WiFi.localIP());

  // Define HTTP Route Handlers
  server.on("/", []() {
    String html = "<html><head><meta name='viewport' content='width=device-width, initial-scale=1'>";
    html += "<style>body{font-family:sans-serif;text-align:center;padding:40px;}";
    html += "button{padding:15px 30px;font-size:18px;margin:10px;cursor:pointer;border-radius:8px;}</style></head>";
    html += "<body><h1>ESP32 Web LED Control</h1>";
    html += "<a href='/on'><button style='background-color:#4CAF50;color:white;'>Turn ON</button></a>";
    html += "<a href='/off'><button style='background-color:#f44336;color:white;'>Turn OFF</button></a>";
    html += "</body></html>";
    server.send(200, "text/html", html);
  });

  server.on("/on", []() {
    digitalWrite(LED_PIN, HIGH);
    server.send(200, "text/html", "<h1>LED is ON</h1><a href='/'><button>BACK</button></a>");
    Serial.println("Command Received -> LED ON");
  });

  server.on("/off", []() {
    digitalWrite(LED_PIN, LOW);
    server.send(200, "text/html", "<h1>LED is OFF</h1><a href='/'><button>BACK</button></a>");
    Serial.println("Command Received -> LED OFF");
  });

  // Start the server
  server.begin();
  Serial.println("HTTP Web Server Started. Navigate to http://" + WiFi.localIP().toString());
}

void loop() {
  // Listen for and handle incoming client HTTP requests
  server.handleClient();
}
```

#### Working Analysis
1. `WiFi.begin()` initiates the 802.11 connection handshake.
2. Once connected, DHCP assigns a private IP address (e.g., `192.168.43.xxx`).
3. `server.handleClient()` polls the TCP stack inside `loop()`, serving incoming HTTP GET requests and triggering GPIO state toggles.

---

### Activity 2: IoT Smart Automatic Curtain / Stage Controller with Web Dashboard

#### Objective
Design an automated theatrical curtain / stage simulation where an IR obstacle sensor detects visitors to automatically open/close the curtain, while an integrated web dashboard provides real-time state visualization and remote LED overrides.

#### Circuit & Pin Configuration
| Component | ESP32 Pin | Logic Level / Note |
| :--- | :--- | :--- |
| **IR Sensor OUT** | GPIO 4 | Active-LOW digital input |
| **Indicator LED** | GPIO 2 | Digital Output |
| **Power (VCC / GND)**| 3.3V / 5V & GND | Common Ground with ESP32 |

#### Source Code
```cpp
/*
 * Experiment 2: IoT Smart Automatic Curtain Controller with Live Web Dashboard
 * Description: Interfacing an IR Obstacle Sensor with ESP32 WebServer.
 *              Serves a theater stage simulation UI stored in PROGMEM.
 *              Uses non-blocking asynchronous AJAX polling for live sensor updates.
 */

#include <WiFi.h>
#include <WebServer.h>

// ---------- Wi-Fi Credentials ----------
const char* ssid     = "navkis";
const char* password = "123456789";

// ---------- Pin Definitions ----------
#define LED_PIN   2
#define IR_SENSOR 4

// Obstacle sensor configuration
#define IR_ACTIVE_LOW true

// Jitter filter: keep "detected" state active for 600 ms after the last trigger
#define HOLD_MS 600

WebServer server(80);

bool ledState = false;
bool objectDetected = false;
unsigned long lastSeen = 0;

// ---------- Web Application Assets (Stored in Flash Memory) ----------
const char webpage[] PROGMEM = R"rawliteral(
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>ESP32 IoT Curtain Controller</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <style>
    :root {
      --night: #14101a;
      --velvet: #7b1626;
      --velvet-dark: #4a0c17;
      --gold: #d9b45a;
      --paper: #f3ead8;
      --muted: #a99f8c;
    }
    * { box-sizing: border-box; }
    body {
      margin: 0;
      min-height: 100vh;
      background: var(--night);
      color: var(--paper);
      font-family: Georgia, "Times New Roman", serif;
      display: flex;
      flex-direction: column;
      align-items: center;
      padding: 28px 16px 40px;
    }
    h1 {
      font-size: 26px;
      font-weight: 600;
      letter-spacing: 0.02em;
      margin: 0 0 20px;
    }
    /* Stage Architecture */
    .stage {
      position: relative;
      width: min(94vw, 560px);
      aspect-ratio: 4 / 3;
      overflow: hidden;
      border: 3px solid var(--gold);
      border-radius: 4px;
      background: #0b0810;
    }
    .backstage {
      position: absolute;
      inset: 0;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      gap: 6px;
      text-align: center;
      background: radial-gradient(ellipse at 50% 40%, #3a2a4a 0%, #0b0810 70%);
    }
    .backstage .big {
      font-size: clamp(24px, 6vw, 40px);
      font-weight: 700;
      color: var(--gold);
    }
    .backstage .small {
      font-size: 15px;
      color: var(--muted);
    }
    .panel {
      position: absolute;
      top: 0;
      bottom: 0;
      width: 51%;
      background:
        repeating-linear-gradient(90deg, rgba(0,0,0,0.38) 0px, rgba(0,0,0,0) 16px, rgba(255,255,255,0.09) 30px, rgba(0,0,0,0) 42px, rgba(0,0,0,0.38) 56px),
        linear-gradient(180deg, var(--velvet) 0%, var(--velvet-dark) 100%);
      transition: transform 1.7s cubic-bezier(0.7, 0, 0.25, 1);
      z-index: 2;
    }
    .panel.left { left: 0; transform-origin: left center; }
    .panel.right { right: 0; transform-origin: right center; }
    .stage.open .panel { transform: scaleX(0.14); }
    .valance {
      position: absolute;
      top: 0;
      left: 0;
      right: 0;
      height: 11%;
      background: linear-gradient(180deg, #5d1020, var(--velvet-dark));
      border-bottom: 4px solid var(--gold);
      z-index: 3;
    }
    .status {
      margin: 22px 0 6px;
      font-size: 20px;
      text-align: center;
    }
    .status b { color: var(--gold); }
    .link {
      font-size: 14px;
      color: var(--muted);
      margin-bottom: 26px;
      min-height: 18px;
    }
    .link.bad { color: #ff8a80; }
    .led {
      display: flex;
      align-items: center;
      gap: 12px;
      flex-wrap: wrap;
      justify-content: center;
    }
    .led span { font-size: 16px; margin-right: 4px; }
    button {
      font-family: inherit;
      font-size: 17px;
      padding: 12px 28px;
      border-radius: 8px;
      border: 2px solid var(--gold);
      background: transparent;
      color: var(--paper);
      cursor: pointer;
      transition: all 0.2s ease;
    }
    button.active {
      background: var(--gold);
      color: var(--night);
      font-weight: 700;
    }
  </style>
</head>
<body>
  <h1>ESP32 Theatrical Curtain Simulation</h1>

  <div class="stage" id="stage">
    <div class="backstage">
      <div class="big">Object Detected</div>
      <div class="small">IR sensor actively triggered</div>
    </div>
    <div class="panel left"></div>
    <div class="panel right"></div>
    <div class="valance"></div>
  </div>

  <p class="status">Curtain state: <b id="curtainText">closed</b></p>
  <div class="link" id="link">Connecting to ESP32...</div>

  <div class="led">
    <span>Stage Lighting: <b id="ledText">OFF</b></span>
    <button id="btnOn"  onclick="setLED('on')">Turn ON</button>
    <button id="btnOff" onclick="setLED('off')" class="active">Turn OFF</button>
  </div>

<script>
  const stage = document.getElementById('stage');
  const curtainText = document.getElementById('curtainText');
  const link = document.getElementById('link');
  const ledText = document.getElementById('ledText');
  const btnOn = document.getElementById('btnOn');
  const btnOff = document.getElementById('btnOff');

  function showLED(isOn) {
    ledText.textContent = isOn ? 'ON' : 'OFF';
    btnOn.classList.toggle('active', isOn);
    btnOff.classList.toggle('active', !isOn);
  }

  function showCurtain(isOpen) {
    stage.classList.toggle('open', isOpen);
    curtainText.textContent = isOpen ? 'OPEN' : 'CLOSED';
  }

  function setLED(state) {
    fetch('/' + state)
      .then(r => r.text())
      .then(t => showLED(t.trim() === 'ON'))
      .catch(() => {});
  }

  function poll() {
    fetch('/status')
      .then(r => r.json())
      .then(d => {
        showCurtain(d.object);
        showLED(d.led);
        link.textContent = 'Telemetry Live';
        link.className = 'link';
      })
      .catch(() => {
        link.textContent = 'Connection Lost. Retrying...';
        link.className = 'link bad';
      })
      .then(() => setTimeout(poll, 300));
  }

  poll();
</script>
</body>
</html>
)rawliteral";

// ---------- HTTP Route Handlers ----------
void handleRoot() {
  server.send(200, "text/html", webpage);
}

void handleON() {
  digitalWrite(LED_PIN, HIGH);
  ledState = true;
  server.send(200, "text/plain", "ON");
  Serial.println("Dashboard Action: LED ON");
}

void handleOFF() {
  digitalWrite(LED_PIN, LOW);
  ledState = false;
  server.send(200, "text/plain", "OFF");
  Serial.println("Dashboard Action: LED OFF");
}

void handleStatus() {
  String json = "{\"object\":" + String(objectDetected ? "true" : "false") +
                ",\"led\":" + String(ledState ? "true" : "false") + "}";
  server.sendHeader("Cache-Control", "no-store");
  server.send(200, "application/json", json);
}

void handleNotFound() {
  server.send(404, "text/plain", "404: Resource Not Found");
}

// ---------- Sensor Filtering & Debouncing ----------
void updateSensor() {
  int raw = digitalRead(IR_SENSOR);
  bool hit = IR_ACTIVE_LOW ? (raw == LOW) : (raw == HIGH);

  if (hit) {
    lastSeen = millis();
  }

  bool now = (lastSeen != 0) && (millis() - lastSeen < HOLD_MS);

  if (now != objectDetected) {
    objectDetected = now;
    Serial.println(objectDetected ? ">> SENSOR TRIGGER: Object Detected (Opening Curtain)"
                                  : ">> SENSOR IDLE: Path Clear (Closing Curtain)");
  }
}

// ---------- Wi-Fi Connection Manager ----------
bool connectWiFi() {
  WiFi.mode(WIFI_STA);
  WiFi.setSleep(false);
  WiFi.begin(ssid, password);

  Serial.print("Connecting to Wi-Fi");
  unsigned long startTime = millis();

  while (WiFi.status() != WL_CONNECTED && (millis() - startTime < 20000)) {
    delay(500);
    Serial.print(".");
  }

  if (WiFi.status() == WL_CONNECTED) {
    Serial.println("\nWi-Fi Connection Established!");
    Serial.print("Access Dashboard at: http://");
    Serial.println(WiFi.localIP());
    return true;
  }

  Serial.println("\nWi-Fi Connection Failed! Please check SSID/Password.");
  return false;
}

void setup() {
  Serial.begin(115200);

  pinMode(LED_PIN, OUTPUT);
  digitalWrite(LED_PIN, LOW);
  pinMode(IR_SENSOR, INPUT);

  if (connectWiFi()) {
    server.on("/", handleRoot);
    server.on("/on", handleON);
    server.on("/off", handleOFF);
    server.on("/status", handleStatus);
    server.onNotFound(handleNotFound);

    server.begin();
    Serial.println("Web Server Ready for Clients.");
  }
}

void loop() {
  updateSensor();

  if (WiFi.status() == WL_CONNECTED) {
    server.handleClient();
  }
}
```

#### Working Analysis
1. **De-jitter Logic (`HOLD_MS = 600`):** Prevents mechanical or optical flutter where the curtain would rapidly open and close if an object hovered near the edge of detection.
2. **`PROGMEM` Memory Storage:** Storing the CSS and HTML template string inside SPI Flash avoids depleting the ESP32's 520 KB SRAM.
3. **Asynchronous Polling:** The browser updates CSS transforms in response to `/status` JSON responses, animating the curtain panels without visual interruption.

---

### Activity 3: Intelligent Distance-Based Motor Speed & Collision Prevention System

#### Objective
Develop an obstacle-aware automated motor control system utilizing an HC-SR04 ultrasonic sensor, L298N H-Bridge driver, visual LED indicators, and an acoustic buzzer. The system implements a three-zone safety hierarchy to modulate speed and avert collisions.

#### Zone Architecture & System Response
```
[ Obstacle ] <--- 7 cm ---> [ WARNING ZONE ] <--- 20 cm ---> [ SAFE ZONE ]
               (DANGER)
          Motor: Full Stop           Motor: Half Speed             Motor: Full Speed
          Buzzer: Continuous ON      Buzzer: OFF                   Buzzer: OFF
          LED: Rapid Flash (100ms)   LED: Slow Flash (500ms)       LED: OFF
```

#### Circuit Interfacing
| Device | Device Pin | ESP32 GPIO | Operating Voltage / Note |
| :--- | :--- | :--- | :--- |
| **L298N Driver** | ENA | GPIO 25 | Hardware PWM Speed Channel (remove ENA jumper) |
| **L298N Driver** | IN1 | GPIO 26 | Direction Control Line A |
| **L298N Driver** | IN2 | GPIO 27 | Direction Control Line B |
| **HC-SR04 Sensor**| TRIG | GPIO 5 | Digital Output (3.3V trigger pulse) |
| **HC-SR04 Sensor**| ECHO | GPIO 18 | Digital Input (**Through $1\text{k}\Omega / 2\text{k}\Omega$ Divider**) |
| **Alert LED** | Anode (+) | GPIO 13 | Series $220\ \Omega$ to GND |
| **Active Buzzer** | Positive (+) | GPIO 14 | Active-HIGH drive |

#### Source Code
```cpp
/*
 * Experiment 3: ESP32 + L298N + HC-SR04 Smart Obstacle Avoidance & Motor Controller
 * Description: Implements a 3-zone safety hierarchy:
 *              - SAFE (> 20 cm)       : Motor operates at full speed; alerts inactive.
 *              - WARNING (7 - 20 cm)  : Motor throttles to slow speed; LED flashes slowly.
 *              - DANGER (<= 7 cm)     : Motor emergency stop; LED flashes rapidly; Buzzer active.
 *              Also executes automated forward/pause/reverse test cycles.
 */

#define IN1 26
#define IN2 27
#define ENA 25  // Hardware PWM speed control pin

#define TRIG_PIN 5
#define ECHO_PIN 18 // Fed through 1k / 2k resistive voltage divider (5V -> 3.3V)

#define LED_PIN    13 // Visual alert indicator
#define BUZZER_PIN 14 // Acoustic warning buzzer

// Threshold boundaries in centimeters
#define WARN_DISTANCE_CM 20
#define STOP_DISTANCE_CM  7

// Speed calibration (8-bit resolution: 0 - 255)
#define FULL_SPEED 255
#define SLOW_SPEED 120

// Hardware PWM parameters
#define PWM_FREQ 1000 // 1 kHz switching frequency
#define PWM_RES     8 // 8-bit resolution (values 0-255)

// Alert flash rates
#define SLOW_BLINK_MS 500
#define FAST_BLINK_MS 100

enum Zone { SAFE, WARNING, DANGER };
Zone zone = SAFE;

// Automated kinematic cycle: Forward (3s) -> Pause (2s) -> Reverse (3s) -> Pause (2s)
const unsigned long phaseDuration[4] = {3000, 2000, 3000, 2000};
int phase = 0;
unsigned long phaseElapsed = 0;

long distance = 999;
unsigned long lastMeasure = 0;
unsigned long lastBlink   = 0;
bool ledState = false;

// Measures obstacle distance using acoustic Time-of-Flight
long getDistanceCm() {
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);
  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);

  // Measure return pulse with a 30 ms timeout (~5 meters max range)
  long duration = pulseIn(ECHO_PIN, HIGH, 30000);
  if (duration == 0) return 999; // Return out-of-range sentinel
  return (duration * 0.0343) / 2;
}

// Sets motor PWM duty cycle with backward compatibility across ESP32 Arduino Core versions
void setMotorSpeed(int speed) {
#if ESP_ARDUINO_VERSION_MAJOR >= 3
  ledcWrite(ENA, speed);
#else
  ledcWrite(0, speed);
#endif
}

void motorStop() {
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, LOW);
  setMotorSpeed(0);
}

void motorForward(int speed) {
  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, LOW);
  setMotorSpeed(speed);
}

void motorReverse(int speed) {
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, HIGH);
  setMotorSpeed(speed);
}

// Determines the operational safety zone based on current distance
void updateZone() {
  Zone newZone;
  if (distance <= STOP_DISTANCE_CM) {
    newZone = DANGER;
  } else if (distance <= WARN_DISTANCE_CM) {
    newZone = WARNING;
  } else {
    newZone = SAFE;
  }

  if (newZone != zone) {
    zone = newZone;
    Serial.print("Proximity: ");
    Serial.print(distance);
    Serial.print(" cm -> Transition to: ");
    if (zone == SAFE)    Serial.println("SAFE ZONE (Full Speed, Alerts Off)");
    if (zone == WARNING) Serial.println("WARNING ZONE (Reduced Speed, Slow Flash)");
    if (zone == DANGER)  Serial.println("DANGER ZONE (Emergency Stop, Fast Flash, Buzzer Active)");
  }
}

// Handles motor kinematic state and overrides speed in case of obstacle
void updateMotor(unsigned long dt) {
  bool isMovingPhase = (phase == 0 || phase == 2);

  if (isMovingPhase) {
    if (zone == DANGER) {
      motorStop(); // Halt motor; kinematic timer paused during obstruction
    } else {
      int speed = (zone == WARNING) ? SLOW_SPEED : FULL_SPEED;
      if (phase == 0) motorForward(speed);
      else            motorReverse(speed);
      phaseElapsed += dt;
    }
  } else {
    motorStop();
    phaseElapsed += dt;
  }

  // Advance kinematic sequence when phase duration expires
  if (phaseElapsed >= phaseDuration[phase]) {
    phase = (phase + 1) % 4;
    phaseElapsed = 0;
    if (phase == 0) Serial.println(">> Motor Kinematics: Moving FORWARD");
    if (phase == 1 || phase == 3) Serial.println(">> Motor Kinematics: PAUSED");
    if (phase == 2) Serial.println(">> Motor Kinematics: Moving REVERSE");
  }
}

// Manages LED flasher and Buzzer alert states asynchronously
void updateAlerts(unsigned long now) {
  if (zone == SAFE) {
    digitalWrite(LED_PIN, LOW);
    digitalWrite(BUZZER_PIN, LOW);
    ledState = false;
    return;
  }

  unsigned long interval = (zone == DANGER) ? FAST_BLINK_MS : SLOW_BLINK_MS;
  if (now - lastBlink >= interval) {
    lastBlink = now;
    ledState = !ledState;
    digitalWrite(LED_PIN, ledState);
  }

  // Sound buzzer continuously only while in DANGER zone
  digitalWrite(BUZZER_PIN, (zone == DANGER) ? HIGH : LOW);
}

void setup() {
  Serial.begin(115200);

  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);
  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);
  pinMode(LED_PIN, OUTPUT);
  pinMode(BUZZER_PIN, OUTPUT);

  // Initialize ESP32 LEDC PWM peripheral
#if ESP_ARDUINO_VERSION_MAJOR >= 3
  ledcAttach(ENA, PWM_FREQ, PWM_RES);
#else
  ledcSetup(0, PWM_FREQ, PWM_RES);
  ledcAttachPin(ENA, 0);
#endif

  motorStop();
  digitalWrite(LED_PIN, LOW);
  digitalWrite(BUZZER_PIN, LOW);

  Serial.println("==================================================");
  Serial.println("ESP32 Smart Collision Prevention Controller Online");
  Serial.println("==================================================");
}

void loop() {
  unsigned long now = millis();

  // Periodic sensor sampling every 60 ms (avoids acoustic echo reflections)
  if (now - lastMeasure >= 60) {
    unsigned long dt = now - lastMeasure;
    lastMeasure = now;

    distance = getDistanceCm();
    updateZone();
    updateMotor(dt);
  }

  updateAlerts(now);
}
```

#### Working Analysis
1. **Echo Timing & Sampling Interval:** HC-SR04 measurements are scheduled every $60\text{ ms}$ using `millis()`. Firing acoustic triggers faster than $50\text{ ms}$ can cause secondary ultrasonic reflections to corrupt distance readings.
2. **Kinematic State Machine:** The motor alternates between forward, pause, and reverse cycles. If an obstacle breaches the $7\text{ cm}$ threshold, the motor halts immediately without advancing the phase timer.
3. **Hardware PWM Core Compatibility:** Code uses conditional compilation (`#if ESP_ARDUINO_VERSION_MAJOR >= 3`) to seamlessly bridge older `ledcSetup` / `ledcAttachPin` APIs with the newer `ledcAttach` API in ESP32 Core v3+.

---

## 5. Summary of Key Learnings & Competencies

- **ESP32 Toolchain Proficiency:** Configured ESP32 boards in the Arduino IDE, managed flash baud rates, and managed high-speed serial monitoring at 115200 bps.
- **Embedded Web Server Engineering:** Built embedded HTTP servers with custom routing, served responsive HTML5/CSS3 graphical interfaces from `PROGMEM`, and implemented JSON REST endpoints.
- **Asynchronous Front-End Telemetry:** Utilized JavaScript `fetch()` to achieve real-time telemetry streaming from microcontroller to web client.
- **Sensor Data Acquisition & Level Shifting:** Acquired data from both active-LOW infrared modules and time-of-flight ultrasonic sensors while protecting 3.3V microcontroller inputs with resistive voltage dividers.
- **Actuator & Power Electronics Interfacing:** Mastered H-Bridge direction control logic and hardware PWM speed modulation via the L298N driver.

---

## 6. Troubleshooting & Technical Considerations

> [!WARNING]
> **Voltage Level Incompatibility (5V vs 3.3V):**  
> The HC-SR04 Echo pin outputs a 5V signal. Directly feeding this into an ESP32 GPIO can damage the chip's internal protection diodes. Always use a $1\text{ k}\Omega : 2\text{ k}\Omega$ resistive voltage divider or bidirectional logic level shifter.

> [!NOTE]
> **Wi-Fi Hotspot Compatibility:**  
> The ESP32 hardware radio supports only **2.4 GHz 802.11 b/g/n** bands. If setting up a smartphone hotspot, ensure the hotspot setting **"Maximize Compatibility"** or **"2.4 GHz Band"** is explicitly enabled.

> [!IMPORTANT]
> **Common Ground Rule for Motor Drivers:**  
> When powering the L298N motor driver using an external battery pack or DC adapter, the **GND** of the external power supply, L298N, and ESP32 **must be tied together**. Without a common ground reference, floating PWM control signals cause erratic motor behavior.

---

## 7. Next Steps & Advanced Project Scope

- [ ] Implementing **MQTT protocol** (e.g., Adafruit IO / HiveMQ) to enable cloud-based telemetry across remote cellular networks.
- [ ] Incorporating **Closed-Loop PID Control** using optical wheel encoders to achieve precise motor RPM and position holding.
- [ ] Replacing the simulated curtain stage with physical **SG90 Micro Servos** or **28BYJ-48 Stepper Motors**.
- [ ] Integrating **OTA (Over-The-Air) Firmware Updates** allowing code to be flashed over Wi-Fi without USB cables.
