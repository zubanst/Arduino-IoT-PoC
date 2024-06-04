# Arduino-IoT-PoC
## Introduction

This is a proof of concept for connected office furniture featuring one RFID RCC522 module connected to a Arduino Nano ESP32 getting commands from a RFID card and broadcasting the command through WiFi to a second Arduino Nano ESP32. The first Arduino Nano ESP32 is the command center and the second is a regular device attached to furniture. The storyboard of the proof of concept is this:
- Register specific RFID card to the RFID control device
- Create WiFi network with Arduino Nano ESP32 as Access point
- Broadcast RFID command through the WiFi network

### Devices for the proof of concept

- Two Arduino Nano ESP32 devices
- One RC522 RFID module
- RFID cards
- LEDs, resistors

### Shematic

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

Go now to the Library manager, and type "rc522" into the search box. Install the "MFRC522 by GithubCommunity" library to be able to use the RFID card.

Select the Arduino Nano ESP32 with the COM port. Under Tools->Port pick from the "Serial ports".

## Programming
### 1. Get the RFID card UID

After having complete the wiring of the Arduino Nano ESP32 with the RFID card, A1 from the schematic, connect the device to the PC with the Arduino IDE. In the IDE, go to File -> Examples -> (Examples from custom libraries) MFRC522 -> DumpInfo, open the sketch and upload it to the device. In the Tools menu open the serial monitor, put your RFID card near the RFID reader and watch in the serial monitor the dump of the 1KB info from the card, including the card's UID. We will need this UID in the next step. This will be the card recognized as allowed to broadcast commands.

The code (DumpInfo.ino):
```
/*
 * --------------------------------------------------------------------------------------------------------------------
 * Example sketch/program showing how to read data from a PICC to serial.
 * --------------------------------------------------------------------------------------------------------------------
 * This is a MFRC522 library example; for further details and other examples see: https://github.com/miguelbalboa/rfid
 * 
 * Example sketch/program showing how to read data from a PICC (that is: a RFID Tag or Card) using a MFRC522 based RFID
 * Reader on the Arduino SPI interface.
 * 
 * When the Arduino and the MFRC522 module are connected (see the pin layout below), load this sketch into Arduino IDE
 * then verify/compile and upload it. To see the output: use Tools, Serial Monitor of the IDE (hit Ctrl+Shft+M). When
 * you present a PICC (that is: a RFID Tag or Card) at reading distance of the MFRC522 Reader/PCD, the serial output
 * will show the ID/UID, type and any data blocks it can read. Note: you may see "Timeout in communication" messages
 * when removing the PICC from reading distance too early.
 * 
 * If your reader supports it, this sketch/program will read all the PICCs presented (that is: multiple tag reading).
 * So if you stack two or more PICCs on top of each other and present them to the reader, it will first output all
 * details of the first and then the next PICC. Note that this may take some time as all data blocks are dumped, so
 * keep the PICCs at reading distance until complete.
 * 
 * @license Released into the public domain.
 * 
 * Typical pin layout used:
 * -----------------------------------------------------------------------------------------
 *             MFRC522      Arduino       Arduino   Arduino    Arduino          Arduino
 *             Reader/PCD   Uno/101       Mega      Nano v3    Leonardo/Micro   Pro Micro
 * Signal      Pin          Pin           Pin       Pin        Pin              Pin
 * -----------------------------------------------------------------------------------------
 * RST/Reset   RST          9             5         D9         RESET/ICSP-5     RST
 * SPI SS      SDA(SS)      10            53        D10        10               10
 * SPI MOSI    MOSI         11 / ICSP-4   51        D11        ICSP-4           16
 * SPI MISO    MISO         12 / ICSP-1   50        D12        ICSP-1           14
 * SPI SCK     SCK          13 / ICSP-3   52        D13        ICSP-3           15
 *
 * More pin layouts for other boards can be found here: https://github.com/miguelbalboa/rfid#pin-layout
 */

#include <SPI.h>
#include <MFRC522.h>

#define RST_PIN         9          // Configurable, see typical pin layout above
#define SS_PIN          10         // Configurable, see typical pin layout above

MFRC522 mfrc522(SS_PIN, RST_PIN);  // Create MFRC522 instance

void setup() {
	Serial.begin(9600);		// Initialize serial communications with the PC
	while (!Serial);		// Do nothing if no serial port is opened (added for Arduinos based on ATMEGA32U4)
	SPI.begin();			// Init SPI bus
	mfrc522.PCD_Init();		// Init MFRC522
	delay(4);				// Optional delay. Some board do need more time after init to be ready, see Readme
	mfrc522.PCD_DumpVersionToSerial();	// Show details of PCD - MFRC522 Card Reader details
	Serial.println(F("Scan PICC to see UID, SAK, type, and data blocks..."));
}

void loop() {
	// Reset the loop if no new card present on the sensor/reader. This saves the entire process when idle.
	if ( ! mfrc522.PICC_IsNewCardPresent()) {
		return;
	}

	// Select one of the cards
	if ( ! mfrc522.PICC_ReadCardSerial()) {
		return;
	}

	// Dump debug info about the card; PICC_HaltA() is automatically called
	mfrc522.PICC_DumpToSerial(&(mfrc522.uid));
}
```

And the output :

```
Card UID: BC DB 2D 17
Card SAK: 08
PICC type: MIFARE 1KB
Sector Block   0  1  2  3   4  5  6  7   8  9 10 11  12 13 14 15  AccessBits
  15     63   00 00 00 00  00 00 FF 07  80 69 FF FF  FF FF FF FF  [ 0 0 1 ] 
         62   00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  [ 0 0 0 ] 
         61   00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  [ 0 0 0 ] 
         60   00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  [ 0 0 0 ] 
...
```
### 2. Program the second Arduino Nano ESP32 as access point

The second Arduino Nano ESP32, the A2 from the schematic will be programmed as Access Point and simple web server, getting commands from the local network. Commands will trigger LED activity as proof for the web call. Wire first the device A2 as showed in the schematic.

The code (SimpleWiFi-Server.ino):

```
/*
 WiFi Web Server with LED Blink

 A simple web server that lets you blink an LED via the web.
 This sketch will print the IP address of your WiFi Shield (once connected)
 to the Serial monitor. From there, you can open that address in a web browser
 to turn on and off the LED on pin 5.

 If the IP address of your shield is yourAddress:
 http://yourAddress/H turns the LED on
 http://yourAddress/L turns it off

 This example is written for a network using WPA2 encryption. For insecure
 WEP or WPA, change the Wifi.begin() call and use Wifi.setMinSecurity() accordingly.

 Circuit:
 * WiFi shield attached
 * LED attached to pin 5

 created for arduino 25 Nov 2012
 by Tom Igoe

ported for sparkfun esp32 
31.01.2017 by Jan Hendrik Berlin
 
 */

#include <WiFi.h>

const char* ssid     = "ArduinoAP_1";
const char* password = "YourSecurePassword";
IPAddress local_IP(192,168,1,10);
IPAddress gateway(192,168,1,1);
IPAddress subnet(255,255,255,0);
WiFiServer server(80);

void setup()
{

    // Start AP

    Serial.begin(115200);
    WiFi.softAPConfig(local_IP, gateway, subnet);
    Serial.println(WiFi.softAP(ssid, password) ? "Ready" : "Fail");
    pinMode(5, OUTPUT);      // set the LED pin mode

    delay(10);

    // We start by displayng connection information

    Serial.println();
    Serial.print("IP address: ");
    Serial.println(WiFi.softAPIP());

    server.begin();

}

void loop(){
 WiFiClient client = server.available();   // listen for incoming clients

  if (client) {                             // if you get a client,
    Serial.println("New Client.");           // print a message out the serial port
    String currentLine = "";                // make a String to hold incoming data from the client
    while (client.connected()) {            // loop while the client's connected
      if (client.available()) {             // if there's bytes to read from the client,
        char c = client.read();             // read a byte, then
        Serial.write(c);                    // print it out the serial monitor
        if (c == '\n') {                    // if the byte is a newline character

          // if the current line is blank, you got two newline characters in a row.
          // that's the end of the client HTTP request, so send a response:
          if (currentLine.length() == 0) {
            // HTTP headers always start with a response code (e.g. HTTP/1.1 200 OK)
            // and a content-type so the client knows what's coming, then a blank line:
            client.println("HTTP/1.1 200 OK");
            client.println("Content-type:text/html");
            client.println();

            // the content of the HTTP response follows the header:
            client.print("Click <a href=\"/H\">here</a> to turn the LED on pin 5 on.<br>");
            client.print("Click <a href=\"/L\">here</a> to turn the LED on pin 5 off.<br>");

            // The HTTP response ends with another blank line:
            client.println();
            // break out of the while loop:
            break;
          } else {    // if you got a newline, then clear currentLine:
            currentLine = "";
          }
        } else if (c != '\r') {  // if you got anything else but a carriage return character,
          currentLine += c;      // add it to the end of the currentLine
        }

        // Check to see if the client request was "GET /H" or "GET /L":
        if (currentLine.endsWith("GET /H")) {
          digitalWrite(5, HIGH);               // GET /H turns the LED on
          delay(2000);
          digitalWrite(5, LOW); 
        }
        if (currentLine.endsWith("GET /L")) {
          digitalWrite(5, LOW);                // GET /L turns the LED off
        }
      }
    }
    // close the connection:
    client.stop();
    Serial.println("Client Disconnected.");
  }
}

```

### 3. Finally program the RFID connected Arduino Nano ESP32 to connect to the AP and send commands

Now we are ready to programm the A1 device from the schematic to connect the previously created AP. When teh selected RFID card with the UID recorded in the first step is presented to the card reader, the green LED of the device assembly will turn on for few seconds and the programm will send the command to the second device A2 by a web call through the WiFi network. If any other RFID card is presented to the RFID card reader, the red LED of the device assembly will turn on for few seconds. No command is sent to the A2 device.

Provided the wiring is already in place, here is the code (RFIDtoWebServer.ino):

```
/*
 * --------------------------------------------------------------------------------------------------------------------
 * Example sketch/program showing how to read data from a PICC to serial and trigger a WebServer
 * --------------------------------------------------------------------------------------------------------------------
 * This is a MFRC522 library example; for further details and other examples see: https://github.com/miguelbalboa/rfid
 * 
 * Example sketch/program showing how to read data from a PICC (that is: a RFID Tag or Card) using a MFRC522 based RFID
 * Reader on the Arduino SPI interface.
 * 
 * When the Arduino and the MFRC522 module are connected (see the pin layout below), load this sketch into Arduino IDE
 * then verify/compile and upload it. To see the output: use Tools, Serial Monitor of the IDE (hit Ctrl+Shft+M). When
 * you present a PICC (that is: a RFID Tag or Card) at reading distance of the MFRC522 Reader/PCD, the serial output
 * will show the ID/UID, type and any data blocks it can read. Note: you may see "Timeout in communication" messages
 * when removing the PICC from reading distance too early.
 * 
 * If your reader supports it, this sketch/program will read all the PICCs presented (that is: multiple tag reading).
 * So if you stack two or more PICCs on top of each other and present them to the reader, it will first output all
 * details of the first and then the next PICC. Note that this may take some time as all data blocks are dumped, so
 * keep the PICCs at reading distance until complete.
 * 
 * @license Released into the public domain.
 * 
 * Typical pin layout used:
 * -----------------------------------------------------------------------------------------
 *             MFRC522      Arduino       Arduino   Arduino    Arduino          Arduino
 *             Reader/PCD   Uno/101       Mega      Nano v3    Leonardo/Micro   Pro Micro
 * Signal      Pin          Pin           Pin       Pin        Pin              Pin
 * -----------------------------------------------------------------------------------------
 * RST/Reset   RST          9             5         D9         RESET/ICSP-5     RST
 * SPI SS      SDA(SS)      10            53        D10        10               10
 * SPI MOSI    MOSI         11 / ICSP-4   51        D11        ICSP-4           16
 * SPI MISO    MISO         12 / ICSP-1   50        D12        ICSP-1           14
 * SPI SCK     SCK          13 / ICSP-3   52        D13        ICSP-3           15
 *
 * More pin layouts for other boards can be found here: https://github.com/miguelbalboa/rfid#pin-layout
 */

#include <SPI.h>
#include <MFRC522.h>
#include <WiFi.h>
#include <HTTPClient.h>

#define RST_PIN         9          // Configurable, see typical pin layout above
#define SS_PIN          10         // Configurable, see typical pin layout above
const char* ssid     = "ArduinoAP_1";
const char* password = "YourSecurePassword";

MFRC522 mfrc522(SS_PIN, RST_PIN);  // Create MFRC522 instance
byte accessUID[4] = {0xBC, 0xDB, 0x2D, 0x17}; // The selected RFID card UID
int greenPin = 2;
int redPin = 3;
void setup() {
  pinMode(greenPin, OUTPUT);
  pinMode(redPin, OUTPUT);
	Serial.begin(115200);		// Initialize serial communications with the PC
	SPI.begin();			// Init SPI bus
	mfrc522.PCD_Init();		// Init MFRC522
	delay(4);				// Optional delay. Some board do need more time after init to be ready, see Readme
	mfrc522.PCD_DumpVersionToSerial();	// Show details of PCD - MFRC522 Card Reader details
	Serial.println(F("Scan PICC to see UID, SAK, type, and data blocks..."));
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
      delay(1000);
      Serial.print(".");
  }

  Serial.println("");
  Serial.println("WiFi connected.");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

void loop() {
  HTTPClient http;
	// Reset the loop if no new card present on the sensor/reader. This saves the entire process when idle.
	if ( ! mfrc522.PICC_IsNewCardPresent()) {
		return;
	}

	// Select one of the cards
	if ( ! mfrc522.PICC_ReadCardSerial()) {
		return;
	}
  
  // check if the selected RFID card UID
  if(mfrc522.uid.uidByte[0] == accessUID[0] && mfrc522.uid.uidByte[1] == accessUID[1] && mfrc522.uid.uidByte[2] == accessUID[2] && mfrc522.uid.uidByte[3] == accessUID[3]) {
    Serial.println("Access granted");
    digitalWrite(greenPin, HIGH);
    http.begin("http://192.168.1.10/H"); //Specify request destination
    http.GET(); //Send the request and trigger the webServer
    http.end();
    delay(2000);
    digitalWrite(greenPin, LOW);
  }else{
    Serial.println("Access denied");
    digitalWrite(redPin, HIGH);
    delay(1000);
    digitalWrite(redPin, LOW);
  }
  mfrc522.PICC_HaltA();
}
```

