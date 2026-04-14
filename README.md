# vermi-city-automated-bin

# Vermi-city: A Micro-Solution to a Macro-Waste Problem 

Vermi-city is an automated, sensor-based monitoring and control system for a vermicomposting unit. It leverages biotechnology and embedded systems to maintain optimal environmental conditions for earthworms, ensuring maximum decomposition efficiency.

Built as a 1st-semester project at **Dayananda Sagar College of Engineering (DSCE)**, Bangalore.

## Problem Statement
Traditional vermicomposting requires labor-intensive manual monitoring. Reliance on manual intervention leads to inconsistent operational conditions, such as improper moisture and temperature levels, which can decrease compost quality or harm the worm colony.

## The Solution: Automated Control
We developed a real-time, closed-loop control system that monitors the environment and autonomously manages actuators to keep parameters within the "Optimal Zone":
* **Temperature:** 25.5°C
* **Moisture:** 60-80% (equivalent to a wrung-out sponge)

## Hardware Architecture

| Component | Role | Details |
| :--- | :--- | :--- |
| **Arduino Uno** | Central Processing Unit | Manages sensor data and actuator logic |
| **DS18B20** | Temperature Sensor | Waterproof probe using OneWire protocol |
| **Capacitive Moisture Sensor** | Moisture Tracking | Analog signal tracking of soil water content |
| **6V Peristaltic Pump** | Irrigation Actuator | Doses water to maintain moisture |
| **12V DC Fan** | Aeration/Cooling Actuator | Maintains optimal temperature |
| **L298N Motor Driver** | Actuator Control | Manages external power for high-current components |
| **16x2 I2C LCD** | User Interface | Displays real-time temperature and moisture data |

### Key Engineering Takeaway: Power Management
A critical challenge in this project was the **Power Rail Distribution**. We utilized an external **12V DC power supply** via a **female DC barrel jack**. 
* **High-Current Rail:** The 12V supply was routed to the L298N driver to power the fan and pump.
* **Logic Rail:** We ensured a shared common ground to protect the Arduino and sensors, preventing the MCU from "burning" during the inductive spikes caused by the peristaltic pump.

## Control Logic (C++)

The system samples data every second and acts based on the following threshold logic:

```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <OneWire.h>
#include <DallasTemperature.h>

// PIN DEFINITIONS
#define ONE_WIRE_BUS 2 // DS18B20 Data wire on Pin 2
const int sensorPin = A0; 
const int fanPin = 7; 
const int pumpPin = 8; 

OneWire oneWire(ONE_WIRE_BUS);
DallasTemperature sensors(&oneWire); 
LiquidCrystal_I2C lcd(0x27, 16, 2);

void setup() {
  lcd.init();
  lcd.backlight();
  sensors.begin();

  pinMode(fanPin, OUTPUT);
  pinMode(pumpPin, OUTPUT);

  lcd.setCursor(0, 0); // Fixed casing (C must be uppercase)
  lcd.print("Greenhouse V1.0");
  delay(2000);
}

void loop() {
  // Read Temperature (DS18B20)
  sensors.requestTemperatures();
  float tempC = sensors.getTempCByIndex(0);

  // Read Soil Moisture (Analog)
  int moistureValue = analogRead(sensorPin);
  int moisturePercent = map(moistureValue, 1023, 0, 0, 100);

  // LCD DISPLAY
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Temp: ");
  lcd.print(tempC, 1); 
  lcd.print("C");
  
  lcd.setCursor(0, 1);
  lcd.print("Soil: ");
  lcd.print(moisturePercent); 
  lcd.print("%");

  // --- ACTUATOR CONTROL LOGIC ---

  // PUMP: Dry soil trigger (< 60%)
  if (moisturePercent < 60) {
    digitalWrite(pumpPin, HIGH); 
  } else {
    digitalWrite(pumpPin, LOW);
  }

  // FAN: High heat trigger (> 25.5°C)
  if (tempC > 25.5) {
    digitalWrite(fanPin, HIGH); 
  } else {
    digitalWrite(fanPin, LOW);
  }

  delay(1000); 
}
```
## Limitations
While the current prototype successfully maintains the environment, there are technical constraints to address in future versions:
* **Manual Calibration:** The soil moisture mapping is currently based on specific laboratory soil conditions; different soil types would require manual re-calibration of the analog thresholds.
* **Power Persistence:** In the event of a power cycle, the system does not currently log "State" history (e.g., how long the pump has been running cumulatively).
* **Local Monitoring Only:** Data visualization is currently limited to the physical 16x2 LCD, requiring physical proximity to check system health.

## Future Outlook
* **IoT Integration (ESP32):** Replacing the Arduino Uno with an ESP32 to push real-time data to a cloud dashboard (Blynk/Thingspeak) for remote monitoring.
* **Machine Learning for Bioconversion:** Implementing ML algorithms to predict the optimal time for compost harvesting based on temperature and moisture trends over time.
* **Solar Integration:** Adding a solar panel and Li-Ion battery management system to make the unit completely off-grid and self-sustaining.
* **pH Level Automation:** Integrating a pH sensor and a secondary dosing pump to maintain a strictly neutral environment (pH 6.5 - 8.0).

## Credits and Acknowledgments
**Project Title**: Vermi-city: A Micro-Solution to a Macro-Waste Problem. 
**Institution**: Dayananda Sagar College of Engineering (DSCE), Bangalore. 
**Department**: Electronics and Communication Engineering. 
**Course**: Innovation and Design Thinking Lab (1BIDTL108)

### The Development Team (Batch A6):
* **Jia Pamelin D souza** (1DS25EC119)
* **Medha T G** (1DS25EC159)
* **Vismaya Vishwas** (1DS25EC301)
* **Shrushti Akki** (1DS25EC254)

**Supervised by**: Dr. Sapna P J

---
