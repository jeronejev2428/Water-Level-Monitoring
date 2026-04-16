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

