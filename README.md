#include <LiquidCrystal.h>

/* ---------------- LCD ---------------- */
LiquidCrystal lcd(12, 11, A1, A2, A3, A4);

/* ---------------- MOTOR (L298) ---------------- */
const int IN1 = 5;
const int IN2 = 6;
const int ENA = 9;

/* ---------------- BUZZER ---------------- */
const int buzzerPin = 4;

/* ---------------- WEIGHT SENSOR (SIMULATED) ---------------- */
const int weightPot = A5;

/* ---------------- FIXED VALUES (DEMO) ---------------- */
const float speedKmph = 15.0;   // fixed demo speed
const int batteryPercent = 80;  // fixed battery

/* ---------------- TIME ---------------- */
const int totalTimeMin = 30;
unsigned long startTime;

/* ---------------- DISPLAY TOGGLE ---------------- */
bool showWeight = true;

void setup() {
  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);
  pinMode(ENA, OUTPUT);
  pinMode(buzzerPin, OUTPUT);

  lcd.begin(16, 2);
  lcd.clear();

  lcd.setCursor(0, 0);
  lcd.print("NEXGEN CART");
  lcd.setCursor(0, 1);
  lcd.print("SYSTEM READY");
  delay(1500);

  startTime = millis();
}

void loop() {

  /* -------- WEIGHT READ -------- */
  int potValue = analogRead(weightPot);
  int weightKg = map(potValue, 0, 1023, 0, 200);

  /* -------- TIME CALC -------- */
  unsigned long elapsedMillis = millis() - startTime;
  int usedMinutes = elapsedMillis / 60000;
  int remainingMinutes = totalTimeMin - usedMinutes;
  if (remainingMinutes < 0) remainingMinutes = 0;

  /* -------- LCD LINE 1 -------- */
  lcd.setCursor(0, 0);
  lcd.print("SPD:");
  lcd.print(speedKmph, 1);
  lcd.print(" B:");
  lcd.print(batteryPercent);
  lcd.print("% ");

  /* -------- OVERLOAD LOGIC -------- */
  if (weightKg > 150) {

    // MOTOR STOP
    digitalWrite(IN1, LOW);
    digitalWrite(IN2, LOW);
    analogWrite(ENA, 0);

    // BUZZER ON
    digitalWrite(buzzerPin, HIGH);

    // LCD ALERT
    lcd.setCursor(0, 1);
    lcd.print("OVERLOAD ALERT ");

  } else {

    // MOTOR RUN
    digitalWrite(IN1, HIGH);
    digitalWrite(IN2, LOW);
    analogWrite(ENA, 120);   // speed for 15 km/h demo

    // BUZZER OFF
    digitalWrite(buzzerPin, LOW);

    // LCD NORMAL INFO
    lcd.setCursor(0, 1);
    if (showWeight) {
      lcd.print("WT:");
      lcd.print(weightKg);
      lcd.print("kg SAFE   ");
    } else {
      lcd.print("LEFT:");
      lcd.print(remainingMinutes);
      lcd.print(" min   ");
    }

    showWeight = !showWeight;
  }

  delay(1000);
}
