source code

#include <LiquidCrystal_I2C.h>

// Инициализация дисплея
LiquidCrystal_I2C lcd(0x27, 16, 2);

// Пины подключения
const int sensorPin = A0;   // Датчик влажности
const int relayPin = 7;     // Реле

// Калибровочные значения (ПОМЕНЯЛИ МЕСТАМИ!)
int dryValue = 320;    // Значение в СУХОЙ почве (маленькое число)
int wetValue = 620;    // Значение во ВЛАЖНОЙ почве (большое число)

int humidityThreshold = 30; // Порог влажности в % (если < 30% - ВКЛЮЧИТЬ насос)

// Переменные для полива
bool isWatering = false;
unsigned long wateringStartTime = 0;
const unsigned long wateringDuration = 5000; // 5 секунд полива

void setup() {
  Serial.begin(9600);
  
  // Инициализация дисплея
  lcd.init();
  lcd.backlight();
  lcd.setCursor(0, 0);
  lcd.print("Auto Watering");
  lcd.setCursor(0, 1);
  lcd.print("System Ready!");
  delay(2000);
  lcd.clear();

  // Настройка реле
  pinMode(relayPin, OUTPUT);
  digitalWrite(relayPin, HIGH); // Выключить реле
  
  Serial.println("System started - FIXED VERSION");
}

void loop() {
  // Чтение датчика
  int sensorValue = analogRead(sensorPin);
  
  // Конвертация в проценты (ПРАВИЛЬНАЯ логика)
  // Чем МЕНЬШЕ значение датчика - тем СУШЕ
  // Чем БОЛЬШЕ значение датчика - тем ВЛАЖНЕЕ
  int humidityPercent = map(sensorValue, dryValue, wetValue, 0, 100);
  humidityPercent = constrain(humidityPercent, 0, 100);
  
  // Вывод на дисплей
  lcd.setCursor(0, 0);
  lcd.print("Moisture: ");
  lcd.print(humidityPercent);
  lcd.print("%  ");
  
  // Отладочная информация в Serial
  Serial.print("Sensor: ");
  Serial.print(sensorValue);
  Serial.print(" | Humidity: ");
  Serial.print(humidityPercent);
  Serial.print("% | Status: ");
  
  // Логика полива (ИСПРАВЛЕННАЯ)
  if (isWatering) {
    lcd.setCursor(0, 1);
    lcd.print("WATERING...    ");
    Serial.println("WATERING");
    
    // Проверяем время полива
    if (millis() - wateringStartTime >= wateringDuration) {
      stopWatering();
      isWatering = false;
      lcd.setCursor(0, 1);
      lcd.print("Watering Done!");
      delay(2000);
    }
  } else {
    // ЕСЛИ влажность МЕНЬШЕ порога - ВКЛЮЧАЕМ полив
    if (humidityPercent > humidityThreshold) {
      lcd.setCursor(0, 1);
      lcd.print("TOO DRY! PUMP ON");
      Serial.println("TOO DRY - STARTING PUMP");
      delay(1000);
      startWatering();
      isWatering = true;
      wateringStartTime = millis();
    } else {
      lcd.setCursor(0, 1);
      lcd.print("Soil OK :)    ");
      Serial.println("OK");
    }
  }
  
  delay(1000);
}

void startWatering() {
  digitalWrite(relayPin, LOW); // Включить реле
  Serial.println("PUMP: ON");
}

void stopWatering() {
  digitalWrite(relayPin, HIGH); // Выключить реле
  Serial.println("PUMP: OFF");
}
