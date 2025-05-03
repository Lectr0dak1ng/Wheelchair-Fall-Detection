Fall Detection System using ESP8266 and MPU6050

This project implements a real-time fall detection system using the ESP8266 NodeMCU and MPU6050 (accelerometer + gyroscope) sensor. When a fall is detected, it triggers an alert via IFTTT webhooks to notify a caregiver or emergency service.

📦 Features
Real-time fall detection using motion and angle thresholds

Three-stage detection mechanism for accurate fall identification

Wi-Fi-based alert system using IFTTT

Low-cost, compact, and wearable

⚙️ Components Used
ESP8266 NodeMCU (Wi-Fi microcontroller)

MPU6050 (3-axis accelerometer + gyroscope)

Breadboard and jumper wires

USB cable for programming

🧠 Fall Detection Logic
Trigger 1: Detects sudden drop in acceleration (Amplitude ≤ 2)

Trigger 2: Checks for strong impact or bounce (Amplitude ≥ 12)

Trigger 3: Verifies stability after impact using angle change (Gyro angle 0–10°)

Fall Confirmed: If all three conditions are met, a fall is registered

The system prevents false positives by using these layered triggers.

📡 Wi-Fi and Alert System
The ESP8266 connects to a local Wi-Fi network.

When a fall is detected, it sends a GET request to the IFTTT Webhook service.

IFTTT can be configured to send an SMS, email, or notification.

🛠️ Setup and Usage
Install Libraries

Wire.h

ESP8266WiFi.h

Wiring

MPU6050 SDA → D2 (GPIO4)

MPU6050 SCL → D1 (GPIO5)

VCC → 3.3V, GND → GND

Edit Credentials

Update the following in the code:

cpp
Copy
Edit
const char *ssid = "YOUR_WIFI_SSID";
const char *pass = "YOUR_WIFI_PASSWORD";
const char *privateKey = "YOUR_IFTTT_WEBHOOK_KEY";
Upload the Code

Using Arduino IDE, select the appropriate board and port.

Flash the code to the ESP8266.

Create IFTTT Applet

Trigger: Webhooks → Receive a web request → Event name: fall_detect

Action: Notification/SMS/email with your custom message
