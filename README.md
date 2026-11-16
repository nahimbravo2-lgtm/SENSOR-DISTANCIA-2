  SENSOR DE DISTANCIA-2
  Sensor de distancia HC-SR04 + Bluetooth HC-05/HC-06 + LEDs
  Compañero del proyecto App Inventor "SensorDistancia.aia"

  - Envía la distancia en cm por Bluetooth (una línea por lectura).
  - Recibe 'C' (CERCA) -> enciende LED ROJO
           'L' (LEJOS) -> enciende LED VERDE
  El umbral se ajusta desde el deslizador de la app.

  CONEXIONES (Arduino UNO/Nano):
    HC-SR04  VCC->5V  GND->GND  TRIG->D9  ECHO->D8
    HC-05    VCC->5V  GND->GND  TXD->D10  RXD->D11 (con divisor 1k/2k a 3.3V)
    LED rojo  D4 -> resistencia 220 ohm -> GND
    LED verde D5 -> resistencia 220 ohm -> GND
*/
#include <SoftwareSerial.h>

const int PIN_TRIG  = 9;
const int PIN_ECHO  = 8;
const int LED_ROJO  = 4;
const int LED_VERDE = 5;

SoftwareSerial BT(10, 11);   // RX, TX del Arduino

unsigned long ultimoEnvio = 0;
const unsigned long INTERVALO_MS = 200;

long medirDistanciaCm() {
  digitalWrite(PIN_TRIG, LOW);
  delayMicroseconds(2);
  digitalWrite(PIN_TRIG, HIGH);
  delayMicroseconds(10);
  digitalWrite(PIN_TRIG, LOW);
  long duracion = pulseIn(PIN_ECHO, HIGH, 30000UL);  // máx ~5 m
  if (duracion == 0) return -1;                       // sin eco
  return duracion / 58;                               // µs -> cm
}

void setup() {
  pinMode(PIN_TRIG, OUTPUT);
  pinMode(PIN_ECHO, INPUT);
  pinMode(LED_ROJO, OUTPUT);
  pinMode(LED_VERDE, OUTPUT);
  Serial.begin(9600);
  BT.begin(9600);          // velocidad por defecto del HC-05/HC-06
}

void loop() {
  // 1) Enviar la distancia periódicamente
  if (millis() - ultimoEnvio >= INTERVALO_MS) {
    ultimoEnvio = millis();
    long d = medirDistanciaCm();
    if (d > 0 && d < 400) {
      BT.println(d);       // termina en '\n' (byte 10) = delimitador en la app
      Serial.println(d);
    }
  }

  // 2) Recibir órdenes de la app
  while (BT.available()) {
    char c = BT.read();
    if (c == 'C') {
      digitalWrite(LED_ROJO, HIGH);
      digitalWrite(LED_VERDE, LOW);
    } else if (c == 'L') {
      digitalWrite(LED_ROJO, LOW);
      digitalWrite(LED_VERDE, HIGH);
    }
  }
}
