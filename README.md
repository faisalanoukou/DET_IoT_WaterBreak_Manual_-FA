# WaterBreak product manual

This is a manual that documents all the steps necessary for setting up the connections for the 'WaterBreak' product.

### Requirements:
- NodeMCU (esp8266)
<img width="auto" height="200" alt="image" src="https://github.com/user-attachments/assets/066a55c7-cfa9-476a-ba51-1e30073f6177" />

- USB-C cable
<img width="200" height="200" alt="image" src="https://github.com/user-attachments/assets/b176f2a5-ad3e-49f8-968b-3e385cfbcb21" />

- Arduino application (v2.3.10)
<img width="200" height="200" alt="image" src="https://github.com/user-attachments/assets/f0aae081-332e-4b93-9bb1-6aceaba0c00a" />

- Push-button
<img width="auto" height="200" alt="image" src="https://github.com/user-attachments/assets/5b7e5de8-8ffb-4f90-a9fd-8a4e460206c8" />

- LED strip
<img width="auto" height="200" alt="image" src="https://github.com/user-attachments/assets/64ab1c26-0070-465c-bacf-6b04bc45f7b9" />

- Vibration motor
<img width="auto" height="200" alt="image" src="https://github.com/user-attachments/assets/9bc82ebd-b142-486e-a9db-83c35140891e" />

- Firebase account
<img width="auto" height="200" alt="image" src="https://github.com/user-attachments/assets/b2c1213f-5a38-4525-99a4-525beb17ad22" />


## Part 1. Firebase database set-up

### 1.1. Firebase website
Navigate to https://console.firebase.google.com/ and create a Firebase account for use.

### 1.2. Database creation
From the navigation bar select 'Build' --> 'Go to build'
<img width="1280" height="629" alt="Screenshot 2026-10-01 140831" src="https://github.com/user-attachments/assets/121cdc14-f0b3-43da-a570-ce23615e4a04" />

Click get started and set up a new Firebase project
<img width="533" height="384" alt="Screenshot 2026-10-01 141743" src="https://github.com/user-attachments/assets/25172faa-4480-4126-ace2-81482a62e12e" />
<img width="620" height="290" alt="Screenshot 2026-10-01 141815" src="https://github.com/user-attachments/assets/ca96f8dd-9149-4dd8-8ed1-eb6810ec8979" />

Name the database with an own fitting name
<img width="722" height="568" alt="Screenshot 2026-10-01 141846" src="https://github.com/user-attachments/assets/d9b7483a-f673-46b2-a466-c46d7efba261" />

Select 'Create Database' to set up the database
<img width="1280" height="628" alt="Screenshot 2026-10-01 142214" src="https://github.com/user-attachments/assets/2d2fd281-1408-4d0d-808d-e0b116216043" />

Import a JSON file from the created database page with the following content
```
{
  "bottles": {
    "waterfles-001": {
      "settings": {
        "dailyGoalMl": 2000,
        "reminderIntervalMinutes": 60
      }
    }
  }
}
```
<img width="944" height="187" alt="Screenshot 2026-10-01 151741" src="https://github.com/user-attachments/assets/f88d49fc-7464-43e6-97a2-9b1bfc57ce6f" />

<br>
<br>
<br>

| Possible mistakes        | Solutions           |
| ------------- |:-------------:|
| Can't upload JSON code      | Create an own JSON file externally and paste the code within this file to upload it |
| Database not public      | Navigate to 'Rules' and adjust the two lines which state 'false' to 'true' and select 'Publish' afterwards   |




## Part 2. Manual API connection WaterBreak

### 2.1. Install Arduino <br>
Navigate to https://www.arduino.cc/en/software/ and download the latest version of the Arduino IDE application.
<img width="910" height="373" alt="image" src="https://github.com/user-attachments/assets/9fe66834-6bec-47fa-acbf-8a895633ada2" />

### 2.2. Install Arduino library
In Arduino open the library section from 'Tools' --> 'Manage libraries'.
Look up and download the following library from this section.
- ArduinoJSON (by Benoit Blanchon)
<img width="221" height="238" alt="image" src="https://github.com/user-attachments/assets/c65379b2-d9fd-4076-bd55-b1ad35c03a52" />

### 2.3. Code for connecting with WiFi (own wifi codes)
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

### 2.4. Connect NodeMCU & check connection
Connect the NodeMCU board to your computer using the usb-C cable.

In Arduino: select the board from 'Tools' --> 'Board' --> 'esp8266' --> 'NodeMCU 1.0 (ESP-12E Module)'.
<img width="728" height="492" alt="Screenshot 2026-10-08 002106" src="https://github.com/user-attachments/assets/d5c7605e-b89b-4be8-aeaf-dffd397a976b" />

Also select the connected port from 'Tools' --> 'Port'.
<img width="349" height="236" alt="image" src="https://github.com/user-attachments/assets/e7c8c239-cbc0-4bfc-8b5d-1c7e1369bf25" />

Upload the code with the arrow button in the top left corner and check for a connection in the Serial Monitor.
<img width="570" height="241" alt="Screenshot 2026-10-01 160932" src="https://github.com/user-attachments/assets/ab07e086-058f-453a-87f2-154018ea1221" />

<br>
<br>
<br>

| Possible mistakes        | Solutions           |
| ------------- |:-------------:|
| No connection in Arduino      | Change WiFi to 2.4 ghz or 'Maximize compatibility' on IOS |
| No WiFi connection      | Change WiFi name/password in the code |
| Upload unsuccessful in Arduino     | Download the (right) library from the library manager   |
| Upload unsuccessful in Arduino     | Select correct board and/or port in Arduino   |




## Part 3. Connecting sensors to NodeMCU

### 3.1. Connect the sensors to the NodeMCU
| Sensor | Sensor pin | NodeMCU pin |
| --- | --- | --- |
| LED strip | GND | GND |
| LED strip | VCC | 3V3 |
| LED strip | Data | D1 |
| Push-button | GND | GND |
| Push-button | VCC | 3V3 |
| Push-button | OUT | D5 |
| Vibration motor | GND | GND |
| Vibration motor | VCC | 3V3 |
| Vibration motor | NC | Don't connect |
| Vibration motor | S16 | D6 |

<img width="auto" height="800" alt="IMG_4096" src="https://github.com/user-attachments/assets/36f2d278-a4e9-4660-8b63-6cce4e1aea8e" />

<br>
<br>
<br>

| Possible mistakes        | Solutions           |
| ------------- |:-------------:|
| Sensors not connected properly      | Double check the correct pin connections on the NodeMCU and the sensors |



## Part 4. Code upload to NodeMCU

### 4.1. Paste the following updated code into Arduino
```
#include <ESP8266WiFi.h>                 // Library: ESP8266WiFi — meegeleverd met ESP8266-boardpakket
#include <ESP8266HTTPClient.h>           // Library: ESP8266HTTPClient — meegeleverd met ESP8266-boardpakket
#include <WiFiClientSecureBearSSL.h>     // Library: WiFiClientSecureBearSSL — meegeleverd met ESP8266-boardpakket
#include <ArduinoJson.h>                 // Library: ArduinoJson by Benoit Blanchon — zelf installeren via Library Manager
#include <Adafruit_NeoPixel.h>           // Library: Adafruit NeoPixel by Adafruit — zelf installeren via Library Manager

const char* WIFI_SSID = "JOUW_WIFI_NAAM";
const char* WIFI_PASSWORD = "JOUW_WIFI_WACHTWOORD";

const char* FIREBASE_URL = "https://waterbreak-iot-default-rtdb.europe-west1.firebasedatabase.app";
const char* BOTTLE_ID = "waterfles-001";

#define LED_PIN      D1
#define NUM_LEDS     12   // Verander dit naar het werkelijke aantal leds op jouw strip.
#define BUTTON_PIN   D5
#define MOTOR_PIN    D6

#define REMINDER_INTERVAL 30000  // 30 seconden in milliseconden.

Adafruit_NeoPixel strip(NUM_LEDS, LED_PIN, NEO_GRB + NEO_KHZ800);

bool reminderActive = false;
bool lastButtonState = HIGH;

unsigned long lastButtonPress = 0;
unsigned long lastDebounceTime = 0;
const unsigned long DEBOUNCE_DELAY = 50;

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
  client.setInsecure();

  HTTPClient https;
  String url = String(FIREBASE_URL) +
    "/bottles/" + BOTTLE_ID + "/settings.json";

  https.begin(client, url);
  int httpCode = https.GET();

  if (httpCode == 200) {
    String response = https.getString();

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
  client.setInsecure();

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

void activateReminder() {
  // Ledstrip aan in blauw.
  for (int i = 0; i < NUM_LEDS; i++) {
    strip.setPixelColor(i, strip.Color(0, 80, 255));
  }
  strip.show();

  // Trilmotor aan.
  digitalWrite(MOTOR_PIN, HIGH);

  reminderActive = true;
  Serial.println("Herinnering actief: ledstrip en trilmotor aan.");
}

void deactivateReminder() {
  // Ledstrip uit.
  strip.clear();
  strip.show();

  // Trilmotor uit.
  digitalWrite(MOTOR_PIN, LOW);

  reminderActive = false;
  Serial.println("Herinnering uitgeschakeld door knopdruk.");
}

void setup() {
  Serial.begin(115200);

  pinMode(BUTTON_PIN, INPUT);
  pinMode(MOTOR_PIN, OUTPUT);
  digitalWrite(MOTOR_PIN, LOW);

  strip.begin();
  strip.clear();
  strip.show();

  connectWiFi();
  readFirebaseSettings();
  writeTestData();

  // Timer starten vanaf het moment dat de NodeMCU opstart.
  lastButtonPress = millis();

  Serial.println("Timer gestart. Herinnering over 30 seconden.");
}

void loop() {
  bool currentButtonState = digitalRead(BUTTON_PIN);

  // Detecteer een knopdruk (overgang van HIGH naar LOW).
  if (currentButtonState == LOW && lastButtonState == HIGH) {
    if (millis() - lastDebounceTime > DEBOUNCE_DELAY) {

      // Herinnering uitschakelen en timer herstarten.
      deactivateReminder();
      lastButtonPress = millis();
      lastDebounceTime = millis();

      Serial.println("Timer herstart. Volgende herinnering over 30 seconden.");
    }
  }

  lastButtonState = currentButtonState;

  // Controleer of de timer van 30 seconden is verstreken.
  if (!reminderActive && millis() - lastButtonPress >= REMINDER_INTERVAL) {
    activateReminder();
  }
}
```

### 4.2. Upload the code in Arduino to the NodeMCU and check the sensors for a successful connection
Press the arrow button in the top left to upload the code
Wait for 30 sec after this for the sensors to react.
After 30 seconds the vibration motor and the LED strip will turn on.
The sensors can be turned off by pressing the push-button.
After pressing the push-button, the timer resets and the vibration motor and the LED will turn on again after 30 seconds.
<img width="auto" height="800" alt="IMG_4097" src="https://github.com/user-attachments/assets/8ef97400-6584-4a70-adee-1b670c71e990" />


<br>
<br>

This process mimics the WaterBreak product by alerting the user of a reminder to drink water after a certain amount of time.

<br>
<br>
<br>

| Possible mistakes        | Solutions           |
| ------------- |:-------------:|
| No WiFi connection      | Turn the router/hotspot on to kickstart the connection (with the right settings) with the NodeMCU |
