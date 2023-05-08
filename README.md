#include <Keypad.h>
#include <BleKeyboard.h>
#include <SPI.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include <Fonts/FreeSansOblique9pt7b.h>




#define SCREEN_WIDTH 128     // OLED display width, in pixels
#define SCREEN_HEIGHT 32     // OLED display height, in pixels
#define OLED_RESET -1        // Reset pin # (or -1 if sharing Arduino reset pin)
#define SCREEN_ADDRESS 0x3C  ///< See datasheet for Address; 0x3D for 128x64, 0x3C for 128x32
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

// Rotary encoder pins
const int encAPin = 27;
const int encBPin = 26;
const int encButtonPin = 25;

int i = 0;
BleKeyboard bleKeyboard;


const byte ROWS = 4;  //four rows
const byte COLS = 4;  //four columns
//define the cymbols on the buttons of the keypads

byte rowPins[ROWS] = { 19, 18, 5, 17 };  //connect to the row pinouts of the keypad
byte colPins[COLS] = { 16, 4, 2, 15 };   //connect to the column pinouts of the keypad


// Keymaps for layers
char keymapLayer1[ROWS][COLS] = {
  { '1', '2', '3', 'A' },
  { '4', '5', '6', 'B' },
  { '7', '8', '9', 'C' },
  { '*', '0', '#', 'D' }
};

char keymapLayer2[ROWS][COLS] = {
  { 'S', 'E', '-', '-' },
  { 'a', 'b', 'c', 'd' },
  { 'e', 'f', 'g', 'h' },
  { 'm', 'n', 'o', 'p' }
};


// Mouse movement constants
const int MOUSE_MOVE_SPEED = 5;
Keypad keypadLayer1 = Keypad(makeKeymap(keymapLayer1), rowPins, colPins, ROWS, COLS);
Keypad keypadLayer2 = Keypad(makeKeymap(keymapLayer2), rowPins, colPins, ROWS, COLS);

// Current layer
int currentLayer = 1;




void setup() {
  Serial.begin(9600);
  if (!display.begin(SSD1306_SWITCHCAPVCC, SCREEN_ADDRESS)) {
    Serial.println(F("SSD1306 allocation failed"));
    for (;;)
      ;  // Don't proceed, loop forever
  }
  Serial.println("Starting BLE work!");
  bleKeyboard.begin();
  pinMode(encAPin, INPUT_PULLUP);
  pinMode(encBPin, INPUT_PULLUP);
  pinMode(encButtonPin, INPUT_PULLUP);
  display.display();
  delay(1000);  // Pause for 2 seconds
  // Clear the buffer
  display.clearDisplay();
}

void loop() {

  char key = getKeypadForCurrentLayer().getKey();
  /* Keypad& keypad = getKeypadForCurrentLayer();
   if (keypad.getNumKeys() > 0) {
     key = keypad.getKey();
   }*/

   Serial.println(key);
  // If button pressed, switch to next layer
  if (digitalRead(encButtonPin) == LOW) {
    currentLayer = (currentLayer + 1) % 2;
    //displayLayerMenu();
    oledTextGoster(60, 30, String(currentLayer));
    delay(250);
  }
  // Read rotary encoder input
  static int lastEncState = HIGH;
  int encState = digitalRead(encAPin);
  if (encState != lastEncState) {
    if (digitalRead(encBPin) != encState) {
      if (key) {
        bleKeyboard.write(key);
        oledTextGoster(0, 30, String(key));
      } else {
        i++;
        oledTextGoster(20, 30, String(i));
      }
    } else {
      if (key) {
        bleKeyboard.write(key);
        oledTextGoster(0, 30, String(key));
      } else {
        i--;
        oledTextGoster(20, 30, String(i));
      }
    }
  }
  lastEncState = encState;
}
Keypad& getKeypadForCurrentLayer() {
  if (currentLayer == 1) {
    return keypadLayer1;
  } else {
    return keypadLayer2;
  }
}
void oledTextGoster(int x, int y, String mesaj) {
  display.clearDisplay();
  display.setTextSize(1);
  display.setFont(&FreeSansOblique9pt7b);
  display.setTextColor(WHITE);
  display.setCursor(x, y);
  display.println(mesaj);
  display.display();
  delay(20);
}
