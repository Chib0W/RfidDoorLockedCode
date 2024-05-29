# RfidDoorLockedCode

#include <SPI.h>
#include <MFRC522.h>
#include <ESP8266WiFi.h>

#define SS_PIN 2   // GPIO2
#define RST_PIN 0  // GPIO0
#define LOCK_PIN 5 // GPIO5

MFRC522 mfrc522(SS_PIN, RST_PIN);
String lastUID = ""; // Variable to store the last UID read
unsigned long lastReadTime = 0; // Variable to store the time of the last card read

void setup() {
  Serial.begin(9600);
  Serial.println("Starting setup...");

  pinMode(LOCK_PIN, OUTPUT);
  digitalWrite(LOCK_PIN, LOW);  // Ensure the lock is initially locked

  SPI.begin();
  mfrc522.PCD_Init();

  Serial.println("Setup complete");
}

void loop() {
  iot_rfid();  // Ensure this function is called in the loop
}

void iot_rfid() {
  if (!mfrc522.PICC_IsNewCardPresent()) {
    return;
  }

  if (!mfrc522.PICC_ReadCardSerial()) {
    return;
  }

  String uid = "";
  for (byte i = 0; i < mfrc522.uid.size; i++) {
    uid += String(mfrc522.uid.uidByte[i] < 0x10 ? "0" : "");
    uid += String(mfrc522.uid.uidByte[i], HEX);
  }

  // Check if the current UID is the same as the last one read
  if (uid == lastUID && millis() - lastReadTime < 5000) {
    // Same UID and within 5 seconds, no need to print it again
    return;
  }

  lastUID = uid; // Update the last UID read
  lastReadTime = millis(); // Update the time of the last card read

  Serial.println("Card UID: " + uid);

  if (uid == "f3eed311") {
    Serial.println("User01 detected");
    openLock();
  } else if (uid == "f3426b0d") {
    Serial.println("User02 detected");
    openLock();
  } else if (uid == "15161718") {
    Serial.println("User03 detected");
    openLock();
  } else {
    // Only print "Unregistered User" if the UID is not recognized
    Serial.println("Unregistered User");
  }

  mfrc522.PICC_HaltA(); // Stop reading
  mfrc522.PCD_StopCrypto1(); // Stop encryption on PCD
}

void openLock() {
  Serial.println("Unlocking...");
  digitalWrite(LOCK_PIN, LOW);  // Unlock the door
  delay(10000);  // Keep it unlocked for 3 seconds
  digitalWrite(LOCK_PIN, HIGH);  // Lock the door again
  Serial.println("Locking...");
}
