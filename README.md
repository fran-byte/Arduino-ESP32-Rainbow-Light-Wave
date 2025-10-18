# Onda de Luz Interactiva con ESP32 – Efecto Arcoíris

Este proyecto crea una **onda de luz que sigue la mano**, usando un **ESP32**, una tira de LEDs WS2812B y un sensor de ultrasonido HC-SR04.  
Con **gradientes arcoíris dinámicos**, rastro largo y suave, ideal para clubs de makers que buscan un efecto realmente impresionante.

---

## **Materiales**

- 1 ESP32 (DevKit v1 o similar)  
- 1 tira de LEDs WS2812B (16–30 LEDs recomendados)  
- 1 sensor de ultrasonido HC-SR04  
- Protoboard y cables  
- Fuente de alimentación 5V para la tira de LEDs  
- Resistencia de 330Ω (opcional, protege la señal de datos)  
- Condensador 1000 µF, 6.3V+ (opcional, suaviza alimentación)  

---

## **Conexión**

### **Tira de LEDs WS2812B**
| Pin del LED | Conexión |
|------------|-----------|
| VCC        | 5V de fuente externa |
| GND        | GND del ESP32 y fuente externa |
| DATA IN    | Pin digital del ESP32 (ejemplo: GPIO 18, con resistencia 330Ω opcional) |

### **Sensor HC-SR04**
| Pin  | Conexión |
|------|-----------|
| VCC  | 5V de fuente externa o VIN del ESP32 |
| GND  | GND |
| TRIG | Pin digital del ESP32 (ejemplo: GPIO 25) |
| ECHO | Pin digital del ESP32 con **divisor de voltaje** a 3.3V |

> ⚠️ Nota: El ECHO del HC-SR04 entrega 5V. Usa un divisor de voltaje o conversor lógico para proteger el ESP32.

---

## **Código de ejemplo con efecto arcoíris**

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

// Medir distancia en cm
long medirDistancia() {
  digitalWrite(TRIG, LOW);
  delayMicroseconds(2);
  digitalWrite(TRIG, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG, LOW);
  long duracion = pulseIn(ECHO, HIGH);
  long distancia = duracion * 0.034 / 2;
  return distancia;
}

// Convertir valor 0-255 a color arcoíris
uint32_t Wheel(byte pos) {
  pos = 255 - pos;
  if(pos < 85) return strip.Color(255 - pos * 3, 0, pos * 3);
  if(pos < 170) { pos -= 85; return strip.Color(0, pos * 3, 255 - pos * 3); }
  pos -= 170;
  return strip.Color(pos * 3, 255 - pos * 3, 0);
}

void loop() {
  long distancia = medirDistancia();
  int ledInicio = map(distancia, 2, 50, 0, NUM_LEDS-1);
  ledInicio = constrain(ledInicio, 0, NUM_LEDS-1);
  waveRainbow(ledInicio);
}

// Onda arcoíris con rastro suave
void waveRainbow(int startLED) {
  for(int i = startLED; i < NUM_LEDS; i++) {
    strip.setPixelColor(i, Wheel((i*256/NUM_LEDS) + millis()/10));
    if(i > 0) strip.setPixelColor(i-1, Wheel((i*256/NUM_LEDS) + millis()/20));
    if(i > 1) strip.setPixelColor(i-2, Wheel((i*256/NUM_LEDS) + millis()/30));
    strip.show();
    delay(30);
    if(i > 3) strip.setPixelColor(i-3, 0); // rastro se desvanece
  }
}
