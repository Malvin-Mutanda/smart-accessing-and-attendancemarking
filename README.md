//the enroll code 
#include <Adafruit_Fingerprint.h>
#include <SoftwareSerial.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

#define ledYellow 7
#define ledRed 6

LiquidCrystal_I2C lcd(0x27, 16, 2);

uint8_t getFingerprintEnroll(uint8_t id);


// pin #2 is IN from sensor (GREEN wire)
// pin #3 is OUT from arduino  (WHITE wire)
SoftwareSerial mySerial(2, 3);

Adafruit_Fingerprint finger = Adafruit_Fingerprint(&mySerial);

void setup()  
{
  lcd.init();
  lcd.backlight();

  pinMode(ledYellow,OUTPUT);
  pinMode(ledRed,OUTPUT);
  
  pinMode(ledYellow, HIGH);
  delay(2000);
  pinMode(ledRed, HIGH);
  delay(1000);  
  
  Serial.begin(9600);
  lcd.setCursor(3, 0);
  lcd.print("fingertest");
  delay(1000);
  digitalWrite(ledRed, LOW);
  delay(3000);
  digitalWrite(ledYellow, HIGH);
  delay(500);
  digitalWrite(ledYellow, LOW);
  delay(500);
  digitalWrite(ledYellow, HIGH);
  delay(500);
  digitalWrite(ledYellow, LOW);
  delay(500);
  digitalWrite(ledYellow, HIGH);
  delay(500);
  
  // set the data rate for the sensor serial port
  finger.begin(57600);
  
  if (finger.verifyPassword()) {
    lcd.setCursor(0,1);
    lcd.print("Sensor Present!");
    delay(2000);

    digitalWrite(ledYellow, HIGH);
    digitalWrite(ledRed, LOW);
    
  } else {
    lcd.setCursor(0,1);
    lcd.print("Sensor Absent");
    delay(2000);
    digitalWrite(ledYellow, LOW);
    digitalWrite(ledRed, HIGH);
    while (1);
  }
}

void loop()                     // run over and over again
{
  lcd.setCursor(0,0);
  lcd.print("Type The ID Here");
  delay(5000);
  lcd.clear();

  digitalWrite(ledYellow, HIGH);
  delay(500);
  digitalWrite(ledYellow, LOW);
  delay(500);
  
  uint8_t id = 0;
  while (true) {
    while (! Serial.available());
    char c = Serial.read();
    if (! isdigit(c)) break;
    id *= 10;
    id += c - '0';
  }
  lcd.setCursor(0,0);
  lcd.print("Enrolling ID #");
  lcd.setCursor(0,1);
  lcd.print(id);
  delay(5000);
  lcd.clear();
  
  while (!  getFingerprintEnroll(id) );
}

uint8_t getFingerprintEnroll(uint8_t id) {
  uint8_t p = -1;
  lcd.setCursor(0,0);
  lcd.print("Enroll Finger");
  lcd.clear();  
  while (p != FINGERPRINT_OK) {
    p = finger.getImage();
    switch (p) {
    case FINGERPRINT_OK:
     lcd.setCursor(0,0);
     lcd.print("Image taken");
     delay(5000);
     lcd.clear();
      break;
    case FINGERPRINT_NOFINGER:
     lcd.setCursor(0,0);
     lcd.print("....Sensing....");
      break;
    case FINGERPRINT_PACKETRECIEVEERR:
      lcd.setCursor(0,0);
      lcd.print("Error Comm");
      break;
    case FINGERPRINT_IMAGEFAIL:
      lcd.setCursor(0,0);
      lcd.print("Imaging error");
      break;
    default:
      lcd.setCursor(0,0);
      lcd.print("Unknown error");
      break;
    }
  }

  // OK success!

  p = finger.image2Tz(1);
  switch (p) {
    case FINGERPRINT_OK:
      lcd.setCursor(0,0);
      lcd.print("Image converted");
      delay(1000);
      lcd.clear();
      break;
    case FINGERPRINT_IMAGEMESS:
      lcd.setCursor(0,0);
      lcd.print("Image too messy");
      return p;
    case FINGERPRINT_PACKETRECIEVEERR:
      lcd.setCursor(0,0);
      lcd.print("Error Comm");
      return p;
    case FINGERPRINT_FEATUREFAIL:
      lcd.setCursor(0,0);
      lcd.print("Could not find fingerprint features");
      return p;
    case FINGERPRINT_INVALIDIMAGE:
      lcd.setCursor(0,0);
      lcd.print("Could not find fingerprint features");
      return p;
    default:
      lcd.setCursor(0,0);
      lcd.print("Unknown error");
      return p;
  }
  lcd.setCursor(0,0);
  lcd.print("Remove finger");
  delay(2000);
  p = 0;
  while (p != FINGERPRINT_NOFINGER) {
    p = finger.getImage();
  }

  p = -1;
  lcd.setCursor(3,0);
  lcd.print("Place same");
  lcd.setCursor(1,1);
  lcd.print("finger again");
  delay(2000);
  lcd.clear();
  while (p != FINGERPRINT_OK) {
    p = finger.getImage();
    switch (p) {
    case FINGERPRINT_OK:
      lcd.setCursor(1,0);
      lcd.print("Image taken");
      delay(5000);
      lcd.clear();
      break;
    case FINGERPRINT_NOFINGER:
      lcd.setCursor(0,0);
      lcd.print("....sensing...");
      delay(5000);
      lcd.clear();
      break;
    case FINGERPRINT_PACKETRECIEVEERR:
      lcd.setCursor(1,0);
      lcd.print("Communication");
      lcd.setCursor(5,1);
      lcd.print("error.");
      delay(5000);
      lcd.clear();
      break;
    case FINGERPRINT_IMAGEFAIL:
      lcd.setCursor(1,0);
      lcd.print("Imaging error");
      delay(5000);
      lcd.clear();
      break;
    default:
      lcd.setCursor(1,0);
      lcd.print("Unknown error");
      delay(5000);
      lcd.clear();
      break;
    }
  }

  // OK success!

  p = finger.image2Tz(2);
  switch (p) {
    case FINGERPRINT_OK:
      lcd.setCursor(0,0);
      lcd.print("Image converted");
      delay(2000);
      lcd.clear();
      break;
    case FINGERPRINT_IMAGEMESS:
      lcd.setCursor(0,0);
      lcd.print("Image too messy");
      delay(5000);
      lcd.clear();
      return p;
    case FINGERPRINT_PACKETRECIEVEERR:
      lcd.setCursor(1,0);
      lcd.print("Communication");
      lcd.setCursor(5,1);
      lcd.print("error.");
      delay(5000);
      lcd.clear();
      return p;
    case FINGERPRINT_FEATUREFAIL:
      Serial.println("Could not find fingerprint features");
      return p;
    case FINGERPRINT_INVALIDIMAGE:
      Serial.println("Could not find fingerprint features");
      return p;
    default:
      lcd.setCursor(1,0);
      lcd.print("Unknown error");
      delay(5000);
      lcd.clear();
      return p;
  }
  
  
  // OK converted!
  p = finger.createModel();
  if (p == FINGERPRINT_OK) {
    lcd.setCursor(0,0);
    lcd.print("Prints matched!");
      delay(3000);
      lcd.clear();

      digitalWrite(ledRed, LOW);
      digitalWrite(ledYellow,HIGH);
      
  } else if (p == FINGERPRINT_PACKETRECIEVEERR) {
      lcd.setCursor(1,0);
      lcd.print("Communication");
      lcd.setCursor(5,1);
      lcd.print("error.");
      delay(5000);
      lcd.clear();
    return p;
  } else if (p == FINGERPRINT_ENROLLMISMATCH) {
    lcd.setCursor(1,0);
    lcd.print("Fingerprints");
    lcd.setCursor(1,1);
    lcd.print("did not match");
    delay(5000);
    lcd.clear();

    digitalWrite(ledYellow, LOW);
    digitalWrite(ledRed, HIGH);
    delay(500);
    digitalWrite(ledRed, LOW);
    delay(500);
    
    return p;
  } else {
      lcd.setCursor(1,0);
      lcd.print("Unknown error");
      delay(5000);
      lcd.clear();
    return p;
  }   
  
  p = finger.storeModel(id);
  if (p == FINGERPRINT_OK) {
    lcd.setCursor(4,0);
    lcd.print("Stored!");
    delay(2000);
    lcd.clear();
    
  } else if (p == FINGERPRINT_PACKETRECIEVEERR) {
      lcd.setCursor(1,0);
      lcd.println("Communication");
      lcd.setCursor(5,1);
      lcd.print("error.");
      delay(5000);
      lcd.clear();     
      
    return p;
  } else if (p == FINGERPRINT_BADLOCATION) {
    Serial.println("Could not store in that location");
    return p;
  } else if (p == FINGERPRINT_FLASHERR) {
    Serial.println("Error writing to flash");
    return p;
  } else {
      lcd.setCursor(1,0);
      lcd.println("Unknown error");
      delay(5000);
      lcd.clear();
    return p;
  }   
}

//smart accessing code 
#include <Adafruit_Fingerprint.h>
#include <LiquidCrystal_I2C.h>
#include <SPI.h>
#include <SoftwareSerial.h>

int getFingerprintIDez();
SoftwareSerial mySerial(2, 3);

#define ledRed 6
#define ledYellow 7

//Created instances
LiquidCrystal_I2C lcd(0x27, 16, 2); 
Adafruit_Fingerprint finger = Adafruit_Fingerprint(&mySerial);

int relayPin = 9;

void setup()
{
 pinMode(ledRed,OUTPUT);
 pinMode(ledYellow,OUTPUT);
 pinMode(relayPin, OUTPUT);
 digitalWrite(relayPin, LOW);
 Serial.begin(9600);
 //while (!Serial);  // For Yun/Leo/Micro/Zero/...
 delay(100);
 
 lcd.backlight();
 lcd.init();
 lcd.setCursor(0, 0);
 lcd.print("Fingerprint Door");
 lcd.setCursor(0, 1);
 lcd.print(" lock by Simblo ");
 
 digitalWrite(ledRed, HIGH);
 digitalWrite(ledYellow, HIGH);
  
 delay(5000);
 lcd.clear();

 // set the data rate for the sensor serial port
 finger.begin(57600);

 if (finger.verifyPassword()) {
   lcd.setCursor(0, 0);
   lcd.print("  FingerPrint ");
   lcd.setCursor(0, 1);
   lcd.print("Sensor Connected");
   delay(3000);
   
   digitalWrite(ledYellow, HIGH);
   delay(1500);
   digitalWrite(ledYellow, LOW);
   delay(1500);
   digitalWrite(ledYellow, HIGH);
   delay(1500);
 }

 else  {
   lcd.setCursor(0, 0);
   lcd.print("Unable to found");
   lcd.setCursor(0, 1);
   lcd.print("Sensor");
   delay(3000);
   lcd.clear();
   lcd.setCursor(0, 0);
   lcd.print("Check Connections");
   delay(3000);

   
   digitalWrite(ledRed, HIGH);
   delay(1000);
   digitalWrite(ledRed, LOW);
   delay(1000);
   digitalWrite(ledRed, HIGH);
   delay(1000);
   digitalWrite(ledRed, LOW);
   delay(1000);
   digitalWrite(ledRed, HIGH);
   delay(1000);   
   digitalWrite(ledYellow, LOW);
   
   while (1) {
     delay(1);
   }
 }
 lcd.clear();
}

void loop()                     // run over and over again
{
 getFingerprintIDez();
 delay(50);            //don't need to run this at full speed.
}

// returns -1 if failed, otherwise returns ID #
int getFingerprintIDez() {
 uint8_t p = finger.getImage();
 if (p != FINGERPRINT_OK)  {
   lcd.setCursor(0, 0);
   lcd.print("  Waiting For");
   lcd.setCursor(0, 1);
   lcd.print("  Valid Finger");

   digitalWrite(ledRed, LOW);
   digitalWrite(ledYellow, HIGH);
   delay(500);
   digitalWrite(ledYellow, LOW);
   delay(500);
   return -1;
 }

 p = finger.image2Tz();
 if (p != FINGERPRINT_OK)  {
   lcd.clear();
   lcd.setCursor(0, 0);
   lcd.print("  Messy Image");
   lcd.setCursor(0, 1);
   lcd.print("  Try Again");
   delay(3000);
   lcd.clear();
   return -1;
 }

 p = finger.fingerFastSearch();
 if (p != FINGERPRINT_OK)  {
   lcd.clear();
   lcd.setCursor(1, 0);
   lcd.print("Access Denied!");
   delay(3000);
   lcd.clear();

   digitalWrite(ledRed, HIGH);
   delay(3000);
   
   return -1;
 }

 // found a match!
 lcd.clear();
 lcd.setCursor(0, 0);
 lcd.print("  Door Unlocked");
 lcd.setCursor(0, 1);
 lcd.print("    Welcome");
 digitalWrite(relayPin, HIGH);
 digitalWrite(ledRed, LOW);
 digitalWrite(ledYellow, HIGH);
 delay(3000);
 digitalWrite(relayPin, LOW);
 lcd.clear();
 return finger.fingerID;
}

