#include "DHT.h"
#include <ESP32Servo.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

#define DHTPIN 15
#define DHTTYPE DHT22
#define LDRPIN 34
#define ROLEPIN 26
#define SERVOPIN 18

#define EKRAN_GENISLIK 128
#define EKRAN_YUKSEKLIK 64

const float ACMA_SICAKLIK = 28.0;
const float KAPAMA_SICAKLIK = 26.0;
const unsigned long OKUMA_ARALIGI = 2000;

DHT dht(DHTPIN, DHTTYPE);
Servo kapak;
Adafruit_SSD1306 ekran(EKRAN_GENISLIK, EKRAN_YUKSEKLIK, &Wire, -1);
bool fanAcik = false;
unsigned long sonOkuma = 0;

void setup() {
  Serial.begin(115200);
  dht.begin();
  pinMode(ROLEPIN, OUTPUT);
  digitalWrite(ROLEPIN, LOW);
  kapak.attach(SERVOPIN);
  kapak.write(0);

  if (!ekran.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {
    Serial.println("OLED bulunamadi!");
    while (true);
  }
  ekran.clearDisplay();
  ekran.setTextSize(1);
  ekran.setTextColor(SSD1306_WHITE);
  ekran.setCursor(0, 0);
  ekran.println("Basliyor...");
  ekran.display();
}

void loop() {
  unsigned long simdi = millis();
  if (simdi - sonOkuma < OKUMA_ARALIGI) {
    return;
  }
  sonOkuma = simdi;

  float sicaklik = dht.readTemperature();
  float nem = dht.readHumidity();
  int isik = analogRead(LDRPIN);

  if (isnan(sicaklik) || isnan(nem)) {
    Serial.println("Okuma hatasi!");
    return;
  }

  if (!fanAcik && sicaklik > ACMA_SICAKLIK) {
    fanAcik = true;
    digitalWrite(ROLEPIN, HIGH);
    kapak.write(90);
  } else if (fanAcik && sicaklik < KAPAMA_SICAKLIK) {
    fanAcik = false;
    digitalWrite(ROLEPIN, LOW);
    kapak.write(0);
  }

  Serial.print("Sicaklik: ");
  Serial.print(sicaklik);
  Serial.print(" C, Nem: ");
  Serial.print(nem);
  Serial.print(", Isik: ");
  Serial.print(isik);
  Serial.print(", Fan: ");
  Serial.println(fanAcik ? "ACIK" : "KAPALI");

  ekran.clearDisplay();
  ekran.setCursor(0, 0);
  ekran.println("AKILLI SERA");

  ekran.setCursor(0, 16);
  ekran.print("Sicaklik: ");
  ekran.print(sicaklik, 1);
  ekran.println(" C");

  ekran.setCursor(0, 28);
  ekran.print("Nem: ");
  ekran.print(nem, 1);
  ekran.println(" %");

  ekran.setCursor(0, 40);
  ekran.print("Isik: ");
  ekran.println(isik);

  ekran.setCursor(0, 52);
  ekran.print("Fan: ");
  ekran.println(fanAcik ? "ACIK" : "KAPALI");

  ekran.display();
}