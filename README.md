# Arduino-IoT-PoC
Introduction

This is a proof of concept for connected office furniture featuring one RFID RCC522 module connected to a Arduino Nano ESP32 getting commands from a RFID card and broadcasting the command through WiFi to a second Arduino Nano ESP32. The first Arduino Nano ESP32 is the command center and the second is a regular device attached to furniture. The storyboard of the proof of concept is this:
- a
- b
- c

## Arduino Nano ESP32

The [Nano ESP32](https://store.arduino.cc/products/nano-esp32) is a powerful addition to the Arduino ecosystem that brings the popular ESP32-S3 to the world of Arduino and MicroPython programming. Whether you're a beginner stepping into the world of IoT or MicroPython, or an advanced user looking to incorporate it into your next product, the Nano ESP32 is the perfect choice. It covers all your needs to kick-start your IoT or MicroPython project like this proof of concept.

### Pinout

![Arduino Nano ESP32 pinout](https://github.com/zubanst/arduino-nano-esp32-pinout.jpg)



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

