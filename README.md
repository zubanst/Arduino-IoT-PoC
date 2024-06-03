# Arduino-IoT-PoC
### Introduction

This is a proof of concept for connected office furniture featuring one RFID RCC522 module connected to a Arduino Nano ESP32 getting commands from a RFID card and broadcasting the command through WiFi to a second Arduino Nano ESP32. The first Arduino Nano ESP32 is the command center and the second is a regular device attached to furniture. The storyboard of the proof of concept is this:
- Register specific RFID card to the RFID control device
- Create WiFi network with Arduino Nano ESP32 as Access point
- Broadcast RFID command through the WiFi network

## Devices for the proof of concept

- Two Arduino Nano ESP32 devices
- One RC522 RFID module
- RFID cards
- LEDs, resistors

## Shematic

The ![schematic](https://github.com/zubanst/Arduino-IoT-PoC/blob/main/IoT-project.pdf) for this proof of concept is attached to the project.

## Details of the devices

### Arduino Nano ESP32

The [Nano ESP32](https://store.arduino.cc/products/nano-esp32) is a powerful addition to the Arduino ecosystem that brings the popular ESP32-S3 to the world of Arduino and MicroPython programming. Whether you're a beginner stepping into the world of IoT or MicroPython, or an advanced user looking to incorporate it into your next product, the Nano ESP32 is the perfect choice. It covers all your needs to kick-start your IoT or MicroPython project like this proof of concept.

#### Pinout

![Arduino Nano ESP32 pinout](https://github.com/zubanst/Arduino-IoT-PoC/blob/main/arduino-nano-esp32-pinout.jpg)

### RC522 RFID Reader/Writer Module

The RFID system has two components: the RFID reader and the tags. They are also called PCD (Proximity Coupling Device) and PICC (Proximity Integrated Circuit Card).

The RFID reader consists of an antenna to emit high-frequency EM waves and a reader/writer. MFRC522 from NXP is an example of such an integrated circuit. Since we are using high-frequency waves in the megahertz range, the size of the antenna can be small.

The RFID tag can be either passive or active. Active tags are powered by batteries while the passive RFID tags are powered by energy from the reader’s interrogating EM waves. The tags are available in different forms or shapes like cards, tags, key forbs, or stickers. Whatever the shape, the RFID tag will consist of an antenna and the RFID chip, which will store all the data. When triggered by an electromagnetic interrogation pulse from a nearby RFID reader, the tag will transmit data back to the reader. The reader will then analyze this data to identify the tag. Unlike a barcode or a QR code, the tag does not need to be within the reader’s line of sight. This makes it easier to process and can be used for tracking objects in closed space.

In this proof of concept we will use a RFID card to broadcast commands to a WiFi connected device. 

#### Pinout

![RC522 RFID pinout](https://github.com/zubanst/Arduino-IoT-PoC/blob/main/RC522-RFID-Reader-Module-Pinout.jpg)

## Arduino IDE

Use Arduino IDE 2.x. Go to the Board Manager, and type "esp32" into the search box. Install "esp32 by Arduino". This is different from "esp32 by Espressif". The Arduino-supplied package includes support for the Arduino Nano ESP32.

Make sure you select the Arduino Nano ESP32 with the DFU port, not the COM port. Under Tools->Port pick from the "dfu ports" list, not from "Serial ports".
Read from a potentiometer and publish over MQTT

The middle pin of the potentiometer is connected to pin A0 on the Nano ESP32.

#include <ArduinoMqttClient.h>
#include <WiFi.h>

#include "arduino_secrets.h"
///////please enter your sensitive data in the Secret tab/arduino_secrets.h
char ssid[] = SECRET_SSID;        // your network SSID (name)
char pass[] = SECRET_PASS;    // your network password (use for WPA, or use as key for WEP)

WiFiClient wifiClient;
MqttClient mqttClient(wifiClient);

const char broker[] = "test.mosquitto.org"; // Address of the MQTT server
int        port     = 1883;
const char topic[]  = "CsHfaWQD";
const char topicA[]  = "CsHfaWQDA";
const char topicC[]  = "CsHfaWQDC";
int counter = 0;
int oldvalue = 0;

void setup() {
  Serial.begin(115200);
  delay(5000);
  pinMode(LED_BUILTIN, OUTPUT);
  pinMode(LED_RED, OUTPUT);
  pinMode(LED_GREEN, OUTPUT);
  pinMode(LED_BLUE, OUTPUT);
  digitalWrite(LED_BUILTIN, LOW);
  digitalWrite(LED_RED, LOW);
  digitalWrite(LED_BLUE, HIGH);
  digitalWrite(LED_GREEN, HIGH);

  // attempt to connect to Wifi network:
  Serial.print("Attempting to connect to WPA SSID: ");
  Serial.println(ssid);
  WiFi.begin(ssid, pass);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("Connected to the network!");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
  Serial.println();

  digitalWrite(LED_RED, HIGH);
  digitalWrite(LED_GREEN, LOW);
  digitalWrite(LED_BLUE, HIGH);

  // You can provide a username and password for authentication
  // mqttClient.setUsernamePassword("user", "password");
  Serial.print("Attempting to connect to the MQTT broker: ");
  Serial.println(broker);

  while (!mqttClient.connect(broker, port)) {
    Serial.print("MQTT connection failed! Error code = ");
    Serial.println(mqttClient.connectError());
  }

  Serial.println("You're connected to the MQTT broker!");
  Serial.println();

  digitalWrite(LED_RED, HIGH);
  digitalWrite(LED_GREEN, HIGH);
  digitalWrite(LED_BLUE, HIGH);
  digitalWrite(LED_BUILTIN, HIGH);
}


// the loop routine runs over and over again forever:
void loop() {
  // read the input on analog pin 0 (which goes from 0 - 4095)
  int sensorValue = analogRead(A0);
  if (abs(sensorValue - oldvalue) > 75) {
    oldvalue = sensorValue;
    Serial.print("Analog: "); Serial.println(sensorValue);
    Serial.print("Counter: "); Serial.println(counter);

    // Text log
    mqttClient.beginMessage(topic);
    mqttClient.print("Analog: "); mqttClient.println(sensorValue);
    mqttClient.print("Counter: "); mqttClient.println(counter);
    mqttClient.endMessage();
    Serial.print("Sent MQTT message for ");
    Serial.println(topic);

    // Potentiometer
    mqttClient.beginMessage(topicA);
    mqttClient.print(sensorValue);
    mqttClient.endMessage();
    Serial.print("Sent MQTT message for ");
    Serial.println(topicA);

    // Counter
    mqttClient.beginMessage(topicC);
    mqttClient.print(counter);
    mqttClient.endMessage();
    Serial.print("Sent MQTT message for ");
    Serial.println(topicC);

    counter++;
    delay(100);
  }
}

You will also need a arduino_secrets.h file like this:

#define SECRET_SSID "MYWIFINET"
#define SECRET_PASS "secretpassword"

