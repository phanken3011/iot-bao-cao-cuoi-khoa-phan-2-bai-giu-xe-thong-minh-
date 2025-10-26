# iot-bao-cao-cuoi-khoa-phan-2-bai-giu-xe-thong-minh-
video hoat dong : https://youtu.be/L61PERvzaVg

Arduino Uno
RFID RC522
Servo SG90 
led
servo

code:
#include <SPI.h>
#include <MFRC522.h>
#include <Servo.h>

#define SS_PIN 10
#define RST_PIN 9
#define LED_XANH 4
#define LED_VANG 7
#define LED_DO 6
#define BUZZER 3  // Còi báo

MFRC522 rfid(SS_PIN, RST_PIN);
Servo servo;

// Hai thẻ RFID hợp lệ
String the1 = "6DD8F55";
String the2 = "215DED5";

bool the1TrongBai = false;
bool the2TrongBai = false;

void setup() {
  Serial.begin(9600);
  SPI.begin();
  rfid.PCD_Init();

  servo.attach(5);
  servo.write(0);

  pinMode(LED_XANH, OUTPUT);
  pinMode(LED_VANG, OUTPUT);
  pinMode(LED_DO, OUTPUT);
  pinMode(BUZZER, OUTPUT);

  capNhatDen();

  Serial.println("=== HE THONG GUI XE RFID (2 CHO - 2 THE + BUZZER) ===");
  Serial.println("Quet the de vao hoac ra bai.");
}

void loop() {
  if (rfid.PICC_IsNewCardPresent() && rfid.PICC_ReadCardSerial()) {
    String mathe = "";
    for (byte i = 0; i < rfid.uid.size; i++) {
      mathe += String(rfid.uid.uidByte[i], HEX);
    }
    mathe.toUpperCase();

    Serial.print("The RFID: ");
    Serial.println(mathe);

    bool theHopLe = false;

    // Xử lý thẻ 1
    if (mathe == the1) {
      theHopLe = true;
      if (!the1TrongBai) {
        the1TrongBai = true;
        Serial.println("Xe 1 DA VAO BAI");
      } else {
        the1TrongBai = false;
        Serial.println("Xe 1 DA RA KHOI BAI");
      }
      servoMoCong();
      baoTheDung();
    }

    // Xử lý thẻ 2
    else if (mathe == the2) {
      theHopLe = true;
      if (!the2TrongBai) {
        the2TrongBai = true;
        Serial.println("Xe 2 DA VAO BAI");
      } else {
        the2TrongBai = false;
        Serial.println("Xe 2 DA RA KHOI BAI");
      }
      servoMoCong();
      baoTheDung();
    }

    // Nếu thẻ sai
    if (!theHopLe) {
      Serial.println("The KHONG HOP LE! ❌");
      baoTheSai();
    }

    capNhatDen();
    rfid.PICC_HaltA();
    delay(1000);
  }
}


void capNhatDen() {
  int soXe = (the1TrongBai ? 1 : 0) + (the2TrongBai ? 1 : 0);

  if (soXe == 0) {
    digitalWrite(LED_XANH, HIGH);
    digitalWrite(LED_VANG, LOW);
    digitalWrite(LED_DO, LOW);
    Serial.println("Trang thai: CON 2 CHO TRONG (DEN XANH)");
  } 
  else if (soXe == 1) {
    digitalWrite(LED_XANH, LOW);
    digitalWrite(LED_VANG, HIGH);
    digitalWrite(LED_DO, LOW);
    Serial.println("Trang thai: CON 1 CHO TRONG (DEN VANG)");
  } 
  else {
    digitalWrite(LED_XANH, LOW);
    digitalWrite(LED_VANG, LOW);
    digitalWrite(LED_DO, HIGH);
    Serial.println("Trang thai: HET CHO (DEN DO)");
  }
}

//  Mở cổng servo
void servoMoCong() {
  servo.write(90);
  delay(2500);
  servo.write(0);
}

//  Kêu 1 lần khi thẻ đúng
void baoTheDung() {
  digitalWrite(BUZZER, HIGH);
  delay(200);
  digitalWrite(BUZZER, LOW);
}

//  Kêu 3 lần khi thẻ sai
void baoTheSai() {
  for (int i = 0; i < 3; i++) {
    digitalWrite(BUZZER, HIGH);
    delay(150);
    digitalWrite(BUZZER, LOW);
    delay(150);
  }
}
