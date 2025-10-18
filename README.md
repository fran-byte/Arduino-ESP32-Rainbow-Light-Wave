# Interactive Rainbow Wave with ESP32

This project creates a **hand-following light wave**, using an **ESP32**, a WS2812B LED strip, and a **3.3V-compatible ultrasonic sensor (HC-SR04-33)**.  
Now with **dynamic rainbow gradients**, smooth trailing, and a visually impressive interactive effect.

---

## **Materials**

- 1 ESP32 (DevKit v1 or similar)  
- 1 WS2812B LED strip (16–30 LEDs recommended)  
- 1 3.3V-compatible ultrasonic sensor (HC-SR04-33)  
- Breadboard and jumper wires  
- 5V power supply for the LED strip  
- 330Ω resistor (optional, protects the data line)  
- 1000 µF, 6.3V+ capacitor (optional, smooths power supply)  

---

## **Wiring**

### **WS2812B LED Strip**
| LED Pin | Connection |
|---------|-----------|
| VCC     | 5V from external power supply |
| GND     | GND of ESP32 and external power |
| DATA IN | Digital pin on ESP32 (e.g., GPIO 18, optional 330Ω resistor) |

### **HC-SR04-33 Ultrasonic Sensor**
| Pin  | Connection |
|------|-----------|
| VCC  | 3.3V from ESP32 |
| GND  | GND |
| TRIG | Digital pin on ESP32 (e.g., GPIO 25) |
| ECHO | Digital pin on ESP32 (e.g., GPIO 26) |

---

## **Example Code with Rainbow Effect**

```cpp
#include <Adafruit_NeoPixel.h>

#define PIN_LED 18
#define NUM_LEDS 30
#define TRIG 25
#define ECHO 26

Adafruit_NeoPixel strip(NUM_LEDS, PIN_LED, NEO_GRB + NEO_KHZ800);

void setup() {
  strip.begin();
  strip.show();
  pinMode(TRIG, OUTPUT);
  pinMode(ECHO, INPUT);
  Serial.begin(115200);
}

// Measure distance in cm
long measureDistance() {
  digitalWrite(TRIG, LOW);
  delayMicroseconds(2);
  digitalWrite(TRIG, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG, LOW);
  long duration = pulseIn(ECHO, HIGH);
  long distance = duration * 0.034 / 2;
  return distance;
}

// Convert 0-255 value to rainbow color
uint32_t Wheel(byte pos) {
  pos = 255 - pos;
  if(pos < 85) return strip.Color(255 - pos * 3, 0, pos * 3);
  if(pos < 170) { pos -= 85; return strip.Color(0, pos * 3, 255 - pos * 3); }
  pos -= 170;
  return strip.Color(pos * 3, 255 - pos * 3, 0);
}

void loop() {
  long distance = measureDistance();
  int ledStart = map(distance, 2, 50, 0, NUM_LEDS-1);
  ledStart = constrain(ledStart, 0, NUM_LEDS-1);
  waveRainbow(ledStart);
}

// Rainbow wave with smooth trailing
void waveRainbow(int startLED) {
  for(int i = startLED; i < NUM_LEDS; i++) {
    strip.setPixelColor(i, Wheel((i*256/NUM_LEDS) + millis()/10));
    if(i > 0) strip.setPixelColor(i-1, Wheel((i*256/NUM_LEDS) + millis()/20));
    if(i > 1) strip.setPixelColor(i-2, Wheel((i*256/NUM_LEDS) + millis()/30));
    strip.show();
    delay(30);
    if(i > 3) strip.setPixelColor(i-3, 0); // fade trailing
  }
}
