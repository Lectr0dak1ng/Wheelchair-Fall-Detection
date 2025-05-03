#include <Wire.h>
#include <ESP8266WiFi.h>
#include <math.h>

const int MPU_addr = 0x68;
int16_t AcX, AcY, AcZ, Tmp, GyX, GyY, GyZ;
float ax = 0, ay = 0, az = 0, gx = 0, gy = 0, gz = 0;
boolean fall = false;
boolean trigger1 = false;
boolean trigger2 = false;
boolean trigger3 = false;
byte trigger1count = 0;
byte trigger2count = 0;
byte trigger3count = 0;
int angleChange = 0;

const char *ssid = "Abinesh"; // 
const char *pass = "123456789"; // 
const char *host = "maker.ifttt.com";
const char *privateKey = "oFGROHYliy901gUUjBZz_";

void send_event(const char *event) ICACHE_FLASH_ATTR;
void mpu_read() ICACHE_FLASH_ATTR;

void setup() {
  Serial.begin(115200);
  Wire.begin();

  Wire.beginTransmission(MPU_addr);
  Wire.write(0x6B);
  Wire.write(0);
  Wire.endTransmission(true);
  Serial.println(F("Wrote to IMU"));

  Serial.println(F("Connecting to Wi-Fi..."));
  WiFi.begin(ssid, pass);
  int wifi_retry = 0;
  while (WiFi.status() != WL_CONNECTED && wifi_retry < 20) {
    delay(500);
    Serial.print(F("."));
    wifi_retry++;
  }

  if (WiFi.status() == WL_CONNECTED) {
    Serial.println(F("\nWi-Fi connected"));
  } else {
    Serial.println(F("\nWi-Fi connection failed"));
  }
}

void loop() {
  mpu_read();

  ax = (AcX - 2050) / 16384.00;
  ay = (AcY - 77) / 16384.00;
  az = (AcZ - 1947) / 16384.00;
  gx = (GyX + 270) / 131.07;
  gy = (GyY - 351) / 131.07;
  gz = (GyZ + 136) / 131.07;

  float Raw_Amp = sqrt(ax * ax + ay * ay + az * az);
  int Amp = Raw_Amp * 10;
  Serial.println(Amp);

  if (Amp <= 2 && !trigger2) {
    trigger1 = true;
    Serial.println(F("TRIGGER 1 ACTIVATED"));
  }

  if (trigger1) {
    trigger1count++;
    if (Amp >= 12) {
      trigger2 = true;
      Serial.println(F("TRIGGER 2 ACTIVATED"));
      trigger1 = false;
      trigger1count = 0;
    }
  }

  if (trigger2) {
    trigger2count++;
    angleChange = sqrt(gx * gx + gy * gy + gz * gz);
    Serial.println(angleChange);
    if (angleChange >= 30 && angleChange <= 400) {
      trigger3 = true;
      trigger2 = false;
      trigger2count = 0;
      Serial.println(F("TRIGGER 3 ACTIVATED"));
    }
  }

  if (trigger3) {
    trigger3count++;
    if (trigger3count >= 10) {
      angleChange = sqrt(gx * gx + gy * gy + gz * gz);
      Serial.println(angleChange);
      if (angleChange >= 0 && angleChange <= 10) {
        fall = true;
        trigger3 = false;
        trigger3count = 0;
        Serial.println(F("FALL DETECTED"));
      } else {
        trigger3 = false;
        trigger3count = 0;
        Serial.println(F("TRIGGER 3 DEACTIVATED"));
      }
    }
  }

  if (fall) {
    send_event("fall_detect");
    fall = false;
  }

  if (trigger2count >= 6) {
    trigger2 = false;
    trigger2count = 0;
    Serial.println(F("TRIGGER 2 DEACTIVATED"));
  }

  if (trigger1count >= 6) {
    trigger1 = false;
    trigger1count = 0;
    Serial.println(F("TRIGGER 1 DEACTIVATED"));
  }

  delay(100);
}

void mpu_read() {
  Wire.beginTransmission(MPU_addr);
  Wire.write(0x3B);
  Wire.endTransmission(false);
  Wire.requestFrom((uint8_t)MPU_addr, (size_t)14, (bool)true);

  AcX = Wire.read() << 8 | Wire.read();
  AcY = Wire.read() << 8 | Wire.read();
  AcZ = Wire.read() << 8 | Wire.read();
  Tmp = Wire.read() << 8 | Wire.read();
  GyX = Wire.read() << 8 | Wire.read();
  GyY = Wire.read() << 8 | Wire.read();
  GyZ = Wire.read() << 8 | Wire.read();
}

void send_event(const char *event) {
  Serial.print(F("Connecting to "));
  Serial.println(host);

  WiFiClient client;
  const int httpPort = 80;

  if (!client.connect(host, httpPort)) {
    Serial.println(F("Connection failed"));
    return;
  }

  String url = String("/trigger/") + event + "/with/key/" + privateKey;

  client.print(F("GET "));
  client.print(url);
  client.print(F(" HTTP/1.1\r\nHost: "));
  client.print(host);
  client.print(F("\r\nConnection: close\r\n\r\n"));

  while (client.connected()) {
    if (client.available()) {
      String line = client.readStringUntil('\r');
      Serial.print(line);
    } else {
      delay(50);
    }
  }

  Serial.println();
  Serial.println(F("Closing connection"));
  client.stop();
}
