# IOT-Dust-Detection-System-in-The-Room
ระบบตรวจจับฝุ่นละอองภายในห้อง จัดทำขึ้นเพื่อพัฒนาระบบตรวจจับฝุ่นละอองภายในห้อง เพื่อนำเสนอการใช้อุปกรณ์ ESP8266 และตัว Dust Sensor ในการแจ้งเตือนภัยจากฝุ่นผ่าน ทาง Line notify 

// Sharp GP2Y1014AU0F Dust Sensor Demo
//
// Board Connection:
//   GP2Y1014    Arduino
//   V-LED       Between R1 and C1
//   LED-GND     C1 and GND
//   LED         Pin 7
//   S-GND       GND
//   Vo          A5
//   Vcc         5V
//
// Serial monitor setting:
//   9600 baud


// Choose program options.
//#define PRINT_RAW_DATA

void Line_Notify(String message) ;
#include <TridentTD_LineNotify.h>
#include <ESP8266WiFi.h> 
#include <WiFiClientSecureAxTLS.h> 
#include <LiquidCrystal_I2C.h>

LiquidCrystal_I2C lcd(0x27,16,2);  
 // set the LCD address to 0x27 for a 16 chars and 2 line display

// Config connect WiFi
#define WIFI_SSID "Karhe"
#define WIFI_PASSWORD "b5940200800"
#define LINE_TOKEN  "HinPSrV7JM6AxT1cgVXcLSQHApDKnbTg9Y8t8e11cZr"

// Line config
void Line_Notify(String Token, String message) ;

#define USE_AVG

// Arduino pin numbers.
const int sharpLEDPin = D0;   // Arduino digital pin 7 connect to sensor LED.
const int sharpVoPin = A0;   // Arduino analog pin 5 connect to sensor Vo.

// For averaging last N raw voltage readings.
#ifdef USE_AVG
#define N 100
static unsigned long VoRawTotal = 0;
static int VoRawCount = 0;
#endif // USE_AVG

// Set the typical output voltage in Volts when there is zero dust. 
static float Voc = 0.6;

// Use the typical sensitivity in units of V per 100ug/m3.
const float K = 0.5;
  

// Helper functions to print a data value to the serial monitor.
void printValue(String text, unsigned int value, bool isLast = false) {
  Serial.print(text);
  Serial.print("=");
  Serial.print(value);
  if (!isLast) {
    Serial.print(", ");
  }
}
void printFValue(String text, float value, String units, bool isLast = false) {
  Serial.print(text);
  Serial.print("=");
  Serial.print(value);
  Serial.print(units);
  if (!isLast) {
    Serial.print(", ");
  }
}



// Arduino setup function.
void setup() {
  // Set LED pin for output.
  pinMode(sharpLEDPin, OUTPUT);
  
  // Start the hardware serial port for the serial monitor.
  Serial.begin(9600);
 lcd.begin();
lcd.backlight();
  // Wait two seconds for startup.
  delay(2000);
  Serial.println("");
  Serial.println("GP2Y1014AU0F Demo");
  Serial.println("=================");
   LINE.setToken(LINE_TOKEN);

}


// Arduino main loop.
void loop() {  
  // Turn on the dust sensor LED by setting digital pin LOW.
  digitalWrite(sharpLEDPin, LOW);

  // Wait 0.28ms before taking a reading of the output voltage as per spec.
  delayMicroseconds(280);

  // Record the output voltage. This operation takes around 100 microseconds.
  int VoRaw = analogRead(sharpVoPin);
  
  // Turn the dust sensor LED off by setting digital pin HIGH.
  digitalWrite(sharpLEDPin, HIGH);

  // Wait for remainder of the 10ms cycle = 10000 - 280 - 100 microseconds.
  delayMicroseconds(9620);
  
  // Print raw voltage value (number from 0 to 1023).
  #ifdef PRINT_RAW_DATA
  printValue("VoRaw", VoRaw, true);
  Serial.println("");
  #endif // PRINT_RAW_DATA
  
  // Use averaging if needed.
  float Vo = VoRaw;
  #ifdef USE_AVG
  VoRawTotal += VoRaw;
  VoRawCount++;
  if ( VoRawCount >= N ) {
    Vo = 1.0 * VoRawTotal / N;
    VoRawCount = 0;
    VoRawTotal = 0;
  } else {
    return;
  }
  #endif // USE_AVG

  // Compute the output voltage in Volts.
  Vo = Vo / 1024.0 * 5.0;
  printFValue("Vo", Vo*1000.0, "mV");

  // Convert to Dust Density in units of ug/m3.
  float dV = Vo - Voc;
  if ( dV < 0 ) {
    dV = 0;
    Voc = Vo;
  }
  float dustDensity = dV / K * 100.0;
  printFValue("DustDensity", dustDensity, "ug/m3", true);
   lcd.setCursor(0,1); 
   lcd.print(dustDensity); 
   lcd.setCursor(4,1); 
   lcd.print("ug/m3"); 
  Serial.println("");


  
   
  if (dustDensity > 200){ 
    lcd.setCursor(0,0); 
   lcd.print("     Danger "); 
    Line_Notify("HinPSrV7JM6AxT1cgVXcLSQHApDKnbTg9Y8t8e11cZr", String(dustDensity));
    Line_Notify("HinPSrV7JM6AxT1cgVXcLSQHApDKnbTg9Y8t8e11cZr", "Danger");
    LINE.notifyPicture("https://scontent.fbkk5-4.fna.fbcdn.net/v/t1.0-9/58461170_1562761500525999_4313654002920194048_n.jpg?_nc_cat=103&_nc_eui2=AeE3EG4NIZS_7YhjWg0k_PJ-PRMsbJ5Kycrrjww-8isDNeuZcqxBTBdr7OJIUkTPhMt8ewPAv8OU5wX1iNo4o3wnZii7KwyP6bslj7RV86CiNA&_nc_ht=scontent.fbkk5-4.fna&oh=724d52b9226d1dad4b4034cf3eb96a2c&oe=5D3A6DDE");
   delay(10000);
   }

   else if (dustDensity >=100){
    Line_Notify("HinPSrV7JM6AxT1cgVXcLSQHApDKnbTg9Y8t8e11cZr", String(dustDensity));
    Line_Notify("HinPSrV7JM6AxT1cgVXcLSQHApDKnbTg9Y8t8e11cZr", "Health risks");
   LINE.notifyPicture("https://scontent.fbkk5-7.fna.fbcdn.net/v/t1.0-9/58380748_1562761530525996_8856840434052235264_n.jpg?_nc_cat=108&_nc_eui2=AeFLMQC4e6j7pUoCWRRCcPefRy_VqinENubo605vU5XR-3vgfb9Hzin8h7Csb1zv0S_XqrAdDe73fFtuokuRFJFbObBmRGJcZUu6UdMwvg2yrg&_nc_ht=scontent.fbkk5-7.fna&oh=29d7467c9fd106f1f6a023a54b3e4c4a&oe=5D3B0F4A");
     lcd.setCursor(0,0); 
    lcd.print("Health risks"); 
    delay(10000);
  }
   else if (dustDensity <= 25){
     lcd.setCursor(0,0); 
    lcd.print(" Very Good  "); 
  }
   else if (dustDensity <= 50){
     lcd.setCursor(0,0); 
    lcd.print("   Good   "); 
  }
    else if (dustDensity <= 100){
     lcd.setCursor(0,0); 
    lcd.print("  Normal    "); 
  }
   
} // END PROGRAM
void Line_Notify(String Token, String message) {
  axTLS::WiFiClientSecure client; // กรณีขึ้น Error ให้ลบ axTLS:: ข้างหน้าทิ้ง

  if (!client.connect("notify-api.line.me", 443)) {
    Serial.println("connection failed");
    return;   
  }

  String req = "";
  req += "POST /api/notify HTTP/1.1\r\n";
  req += "Host: notify-api.line.me\r\n";
  req += "Authorization: Bearer " + String(Token) + "\r\n";
  req += "Cache-Control: no-cache\r\n";
  req += "User-Agent: ESP8266\r\n";
  req += "Connection: close\r\n";
  req += "Content-Type: application/x-www-form-urlencoded\r\n";
  req += "Content-Length: " + String(String("message=" + message).length()) + "\r\n";
  req += "\r\n";
  req += "message=" + message;
  // Serial.println(req);
  client.print(req);
    
  delay(20);

  // Serial.println("-------------");
  while(client.connected()) {
    String line = client.readStringUntil('\n');
    if (line == "\r") {
      break;
    }
    //Serial.println(line);
  }
  // Serial.println("-------------");
}
