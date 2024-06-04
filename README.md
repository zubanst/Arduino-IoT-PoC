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
### Get the RFID card UID

After having complete the wiring of the Arduino Nano ESP32 A1 with the RFID card from the schematic, connect the device to the PC with the Arduino IDE. In the IDE, go to File -> Examples -> (Examples from custom libraries) MFRC522 -> DumpInfo, open the sketch and upload it to the device. In the Tools menu open the serial monitor, put your RFID card near the RFID reader and watch in the serial monitor the dump of the 1KB info from the card, including the card's UID. We will need this UID in the next step. This will be the card recognized as allowed to broadcast commands.
The code:
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


#define SECRET_PASS "secretpassword"

