# Manual API connection WaterBreak

In this manual I will be explaining how I made a connection with an external API using my NodeMCU board and a Firebase server.

## Requirements:
- NodeMCU (esp8266)
<img width="450" height="300" alt="image" src="https://github.com/user-attachments/assets/066a55c7-cfa9-476a-ba51-1e30073f6177" />

- Usb-C cable
<img width="200" height="200" alt="image" src="https://github.com/user-attachments/assets/b176f2a5-ad3e-49f8-968b-3e385cfbcb21" />

- Arduino application (v2.3.10)
<img width="200" height="200" alt="image" src="https://github.com/user-attachments/assets/f0aae081-332e-4b93-9bb1-6aceaba0c00a" />


## 1. Install Arduino <br>
Navigate to https://www.arduino.cc/en/software/ and download the latest version of the Arduino IDE application.
<img width="910" height="373" alt="image" src="https://github.com/user-attachments/assets/9fe66834-6bec-47fa-acbf-8a895633ada2" />


## 3. Install Arduino library
In Arduino open the library section from 'Tools' --> 'Manage libraries'.
Look up and download the following library from this section.
- ArduinoJSON (by Benoit Blanchon)
<img width="221" height="238" alt="image" src="https://github.com/user-attachments/assets/c65379b2-d9fd-4076-bd55-b1ad35c03a52" />

## 4. Code for connecting with WiFi (own wifi codes)
Copy the following code within the Arduino file:
```
#include <ESP8266WiFi.h>                 // Library: ESP8266WiFi — meegeleverd met ESP8266-boardpakket
#include <ESP8266HTTPClient.h>           // Library: ESP8266HTTPClient — meegeleverd met ESP8266-boardpakket
#include <WiFiClientSecureBearSSL.h>     // Library: WiFiClientSecureBearSSL — meegeleverd met ESP8266-boardpakket
#include <ArduinoJson.h>                 // Library: ArduinoJson by Benoit Blanchon — zelf installeren via Library Manager

const char* WIFI_SSID = "JOUW_WIFI_NAAM";
const char* WIFI_PASSWORD = "JOUW_WIFI_WACHTWOORD";

// Plak hier de volledige Firebase-URL, inclusief https://
const char* FIREBASE_URL = "https://waterbreak-iot-default-rtdb.europe-west1.firebasedatabase.app/";
const char* BOTTLE_ID = "waterfles-001";

void connectWiFi() {
  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  Serial.print("Verbinden met wifi");

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println();
  Serial.println("Wifi verbonden.");
}

void readFirebaseSettings() {
  BearSSL::WiFiClientSecure client;
  client.setInsecure(); // Alleen voor dit prototype.

  HTTPClient https;
  String url = String(FIREBASE_URL) +
    "/bottles/" + BOTTLE_ID + "/settings.json";

  https.begin(client, url);
  int httpCode = https.GET();

  if (httpCode == 200) {
    String response = https.getString();

    // ArduinoJson versie 7
    JsonDocument document;
    DeserializationError error = deserializeJson(document, response);

    if (!error) {
      int dailyGoalMl = document["dailyGoalMl"] | 2000;
      int reminderIntervalMinutes = document["reminderIntervalMinutes"] | 60;

      Serial.print("Dagdoel: ");
      Serial.print(dailyGoalMl);
      Serial.println(" ml");

      Serial.print("Herinnering na: ");
      Serial.print(reminderIntervalMinutes);
      Serial.println(" minuten");
    } else {
      Serial.println("Firebase-data bevat geen geldige JSON.");
    }
  } else {
    Serial.print("Fout bij Firebase. HTTP-code: ");
    Serial.println(httpCode);
  }

  https.end();
}

void writeTestData() {
  BearSSL::WiFiClientSecure client;
  client.setInsecure(); // Alleen voor dit prototype.

  HTTPClient https;
  String url = String(FIREBASE_URL) +
    "/bottles/" + BOTTLE_ID + "/current.json";

  String json = "{\"waterRemainingMl\":600,\"message\":\"Test vanaf NodeMCU\"}";

  https.begin(client, url);
  https.addHeader("Content-Type", "application/json");

  int httpCode = https.PUT(json);

  Serial.print("Data verstuurd. HTTP-code: ");
  Serial.println(httpCode);

  https.end();
}

void setup() {
  Serial.begin(115200);
  connectWiFi();
  readFirebaseSettings();
  writeTestData();
}

void loop() {
  // Voor deze test hoeft hier niets te staan.
}
```
!! Make sure to change the WiFi name and password for your the connection that you are using for this.
- Change 'JOUW_WIFI_NAAM' to the name of the WiFi connection.
- Change 'JOUW_WIFI_WACHTWOORD' to the password of the WiFi connection.

## 5. Connect NodeMCU & check connection
Connect the NodeMCU board to your computer using the usb-C cable.

In Arduino: select the board from 'Tools' --> 'Board' --> 'esp8266' --> 'NodeMCU 1.0 (ESP-12E Module)'.
<img width="728" height="492" alt="Screenshot 2026-10-08 002106" src="https://github.com/user-attachments/assets/d5c7605e-b89b-4be8-aeaf-dffd397a976b" />

Also select the connected port from 'Tools' --> 'Port'.
<img width="349" height="236" alt="image" src="https://github.com/user-attachments/assets/e7c8c239-cbc0-4bfc-8b5d-1c7e1369bf25" />

Upload the code with the arrow button in the top left corner and check for a connection in the Serial Monitoer.
