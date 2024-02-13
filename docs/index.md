# The TardiSat Project

## Satellite LoRaWAN TTN gateway
TardiSat is a project for a picosatellite that acts as a gateway to The Things Network and as such can connect to the Internet and provide (low-speed, low-bandwidth) Internet coverage in rural areas.

### Alternatives: Meshtastic
Meshtastic is being evaluated as an alternative, but compared to TTN there are no range tests that have gone past 500 km, so we can't in good faith rely on solid technology in unproven areas.

## LoRa Modulation
You can see a detailed description of how the LoRa physical layer works in the [following article](https://wirelesspi.com/understanding-lora-phy-long-range-physical-layer/)

## Hardware
### LILYGO board
Currently, we are prototyping with two LYLYGO T3_V1.6.1 modules with the following pinout:
![LILYGO board pinout](lora32_v1.6.1_pinmap.jpg)

You can learn more about this board and others from the same series on the [official GitHub repository](https://github.com/Xinyuan-LilyGO/LilyGo-LoRa-Series/tree/master).

## Software
We recently changed to PlatformIO for prototyping in a more advanced development environment.
Currently, we have tested simplex communication between both boards with the following code: 
``` cpp title="rx_test.cpp" linenums="1"
#include <Arduino.h>
#include <SPI.h>
#include <LoRa.h>

#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

#define SCK 5
#define MISO 19
#define MOSI 27
#define SS 18
#define RST 23 //si no funciona prueba con 14 y si no 12
#define DIO0 26

#define BAND 868E6

#define OLED_SDA 4
#define OLED_SCL 15 
#define SCREEN_WIDTH 128 // OLED display width, in pixels
#define SCREEN_HEIGHT 64 // OLED display height, in pixels

Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RST);

String LoRaData;

void setup() { 
  //initialize Serial Monitor
  Serial.begin(115200);
  
  //reset OLED display via software
  pinMode(OLED_RST, OUTPUT);
  digitalWrite(OLED_RST, LOW);
  delay(20);
  digitalWrite(OLED_RST, HIGH);
  
  //initialize OLED
  Wire.begin(OLED_SDA, OLED_SCL);
  if(!display.begin(SSD1306_SWITCHCAPVCC, 0x3c, false, false)) { // Address 0x3C for 128x32
    Serial.println(F("Asignacion a SSD1306 fallida"));
    for(;;); // Don't proceed, loop forever
  }

  display.clearDisplay();
  display.setTextColor(WHITE);
  display.setTextSize(1);
  display.setCursor(0,0);
  display.print("LORA RECEIVER ");
  display.display();

  Serial.println("LoRa Receiver Test");
  
  //SPI LoRa pins
  SPI.begin(SCK, MISO, MOSI, SS);
  //setup LoRa transceiver module
  LoRa.setPins(SS, RST, DIO0);

  if (!LoRa.begin(BAND)) {
    Serial.println("Fallo al inicializar LoRa!");
    while (1);
  }
  Serial.println("LoRa inicializacion OK!");
  display.setCursor(0,10);
  display.println("LoRa inicializacion OK!");
  display.display();  
}

void loop() {

  //try to parse packet
  int packetSize = LoRa.parsePacket();
  if (packetSize) {
    //received a packet
    Serial.print("Recibido el paquete ");

    //read packet
    while (LoRa.available()) {
      LoRaData = LoRa.readString();
      Serial.print(LoRaData);
    }

    //print RSSI of packet
    int rssi = LoRa.packetRssi();
    Serial.print(" con RSSI ");    
    Serial.println(rssi);

   // Dsiplay information
   display.clearDisplay();
   display.setCursor(0,0);
   display.print("LORA RECEIVER");
   display.setCursor(0,20);
   display.print("Recibido el paquete:");
   display.setCursor(0,30);
   display.print(LoRaData);
   display.setCursor(0,40);
   display.print("RSSI:");
   display.setCursor(30,40);
   display.print(rssi);
   display.display();   
  }
}
```

``` cpp title="tx_test.cpp" linenums="1"
#include <SPI.h>
#include <LoRa.h>

#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

#define SCK 5
#define MISO 19
#define MOSI 27
#define SS 18
#define RST 23
#define DIO0 26

#define BAND 868E6

#define OLED_SDA 21
#define OLED_SCL 22 
#define SCREEN_WIDTH 128 // OLED display width, in pixels
#define SCREEN_HEIGHT 64 // OLED display height, in pixels

//packet counter
int counter = 0;

Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RST);

void setup() {
  Serial.begin(115200);

  //reset OLED display via software
  pinMode(OLED_RST, OUTPUT);
  digitalWrite(OLED_RST, LOW);
  delay(20);
  digitalWrite(OLED_RST, HIGH);

  //initialize OLED
  Wire.begin(OLED_SDA, OLED_SCL);
  if(!display.begin(SSD1306_SWITCHCAPVCC, 0x3c, false, false)) { // Address 0x3C for 128x32
    Serial.println(F("asignacion a SSD1306 fallida"));
    for(;;); // Don't proceed, loop forever
  }
  
  display.clearDisplay();
  display.setTextColor(WHITE);
  display.setTextSize(1);
  display.setCursor(0,0);
  display.print("LORA SENDER ");
  display.display();
  
  Serial.println("LoRa Sender Test");

  //SPI LoRa pins
  SPI.begin(SCK, MISO, MOSI, SS);
  //setup LoRa transceiver module
  LoRa.setPins(SS, RST, DIO0);
  
  if (!LoRa.begin(BAND)) {
    Serial.println("Fallo al inicializar LoRa!");
    while (1);
  }
  Serial.println("LoRa inicializacion OK!");
  display.setCursor(0,10);
  display.print("LoRa inicializacion OK!");
  display.display();
  delay(2000);
}

void loop() {
   
  Serial.print("Enviando paquete: ");
  Serial.println(counter);

  //Send LoRa packet to receiver
  LoRa.beginPacket();
  LoRa.print("hola ");
  LoRa.print(counter);
  LoRa.endPacket();
  
  display.clearDisplay();
  display.setCursor(0,0);
  display.println("LORA SENDER");
  display.setCursor(0,20);
  display.setTextSize(1);
  display.print("paquete LoRa enviado.");
  display.setCursor(0,30);
  display.print("Contador:");
  display.setCursor(50,30);
  display.print(counter);      
  display.display();

  counter++;
  
  delay(10000);
}
```

### PlatformIO setup
### LoRaWAN TTN integration
### Testing suite
#### Unit tests
#### Integration tests
#### End-to-end test
