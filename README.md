# Water-Level-Monitoring


A real-time water level monitoring system that uses an ultrasonic sensor to measure water depth and displays the results on an LCD screen. Powered by the ESP8266, this project is ready for IoT integration.

## 📝 Project Description
This project measures the distance between the sensor (placed at the top of a tank) and the water surface. The ESP8266 processes this distance to calculate the percentage of water remaining. The data is updated in real-time on a 16x2 LCD display using the I2C protocol to minimize wiring.

## 🛠 Components Used
* **ESP8266 (NodeMCU):** The brain of the project, providing processing power and Wi-Fi capability.
* **HC-SR04 Ultrasonic Sensor:** Measures distance using sound waves.
* **16x2 LCD with I2C Module:** Displays the level and status without using many GPIO pins.
* **Jumper Wires & Breadboard:** For hardware prototyping.

## 🔌 Wiring Connections

### 1. HC-SR04 Ultrasonic Sensor
| HC-SR04 Pin | ESP8266 (NodeMCU) Pin |
| :--- | :--- |
| VCC | Vin (5V) |
| GND | GND |
| Trig | D7 (GPIO 14) |
| Echo | D8 (GPIO 12) |

### 2. I2C LCD Display
| I2C Module Pin | ESP8266 (NodeMCU) Pin |
| :--- | :--- |
| VCC | Vin (5V) |
| GND | GND |
| SDA | D2 (GPIO 4) |
| SCL | D1 (GPIO 5) |

## Water Level Monitor Code

This project uses an ESP8266, an ultrasonic sensor, and Blynk to monitor water levels and control a pump.

```cpp
#define BLYNK_TEMPLATE_ID "TMPL3TLEJ4abu"
#define BLYNK_TEMPLATE_NAME "water level2"
#define BLYNK_AUTH_TOKEN "EDrBlg3236F-2iUCFW68_SxUAWv21KJw"

#include <ESP8266WiFi.h>
#include <BlynkSimpleEsp8266.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

// --- WiFi ---
char ssid[] = "Jeron";
char pass[] = "12345678";

// --- Pin Definitions ---
#define TRIG_PIN D6
#define ECHO_PIN D7
#define RELAY_PIN D5

// --- I2C LCD ---
LiquidCrystal_I2C lcd(0x27, 16, 2);

// --- Tank Calibration ---
#define TANK_EMPTY_CM 25.0
#define TANK_FULL_CM 5.0

// --- Pump Logic (UPDATED) ---
#define PUMP_ON_LEVEL 88   // ON below 88%
#define PUMP_OFF_LEVEL 90  // OFF at 90%

BlynkTimer timer;
bool pumpState = false;
bool manualMode = false;

// -----------------------------------------------
float getDistanceCM() {
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);

  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);

  long dur = pulseIn(ECHO_PIN, HIGH, 30000);

  if (dur == 0) return -1;
  return (dur * 0.0343) / 2.0;
}

// Improved level calculation
int getLevel(float dist) {
  if (dist < 0) return -1;
  if (dist > TANK_EMPTY_CM) dist = TANK_EMPTY_CM;
  if (dist < TANK_FULL_CM) dist = TANK_FULL_CM;
  float pct = ((TANK_EMPTY_CM - dist) / (TANK_EMPTY_CM - TANK_FULL_CM)) * 100.0;
  return constrain((int)pct, 0, 100);
}

void setPump(bool on) {
  pumpState = on;
  digitalWrite(RELAY_PIN, on ? LOW : HIGH); // active-low relay
  Blynk.virtualWrite(V1, on ? 255 : 0);
}

// --- Manual Mode ---
BLYNK_WRITE(V4) {
  manualMode = param.asInt();
}

// --- Manual Pump Control ---
BLYNK_WRITE(V2) {
  if (manualMode) {
    setPump(param.asInt());
  }
}

// -----------------------------------------------
void updateLCD(int level) {
  lcd.setCursor(0, 0);
  lcd.print("Level:");
  
  if (level < 10) lcd.print("  ");
  else if (level < 100) lcd.print(" ");
  
  lcd.print(level);
  lcd.print("%");

  lcd.setCursor(12, 0);
  if (level <= 10)      lcd.print(" LOW");
  else if (level >= 90) lcd.print("FULL");
  else                  lcd.print("  OK");

  lcd.setCursor(0, 1);
  lcd.print("Pump:");
  lcd.print(pumpState ? "ON " : "OFF");
  lcd.print(manualMode ? " [MAN]" : " [AUTO]");
}

// -----------------------------------------------
void monitorWater() {
  float dist = getDistanceCM();
  int level = getLevel(dist);

  // Debug
  Serial.print("Distance: ");
  Serial.print(dist);
  Serial.print(" cm | Level: ");
  Serial.println(level);

  if (level < 0) {
    lcd.setCursor(0, 0);
    lcd.print("Sensor Error!   ");
    lcd.setCursor(0, 1);
    lcd.print("Check wiring    ");
    return;
  }

  updateLCD(level);
  Blynk.virtualWrite(V0, level);

  // AUTO MODE
  if (!manualMode) {
    if (level <= PUMP_ON_LEVEL && !pumpState) {
      setPump(true);
      Blynk.logEvent("pump_on", "Pump ON (Below 90%)");
    }
    else if (level >= PUMP_OFF_LEVEL && pumpState) {
      setPump(false);
      Blynk.logEvent("pump_off", "Pump OFF (Tank Full)");
    }
  }

  // Alerts
  if (level <= 10)       Blynk.virtualWrite(V3, "CRITICAL: Low!");
  else if (level >= 95)  Blynk.virtualWrite(V3, "Tank Full!");
  else                   Blynk.virtualWrite(V3, "OK");
}

// ===================== SETUP =====================
void setup() {
  Serial.begin(115200);

  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);
  pinMode(RELAY_PIN, OUTPUT);
  digitalWrite(RELAY_PIN, HIGH); // Pump OFF

  Wire.begin(D2, D1);

  lcd.init();
  lcd.backlight();
  lcd.setCursor(0, 0);
  lcd.print("Water Monitor");
  lcd.setCursor(0, 1);
  lcd.print("Connecting...");

  Blynk.begin(BLYNK_AUTH_TOKEN, ssid, pass);

  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Blynk Connected");
  delay(1500);
  lcd.clear();

  timer.setInterval(2000L, monitorWater);
}

// ===================== LOOP =====================
void loop() {
  Blynk.run();
  timer.run();
}
