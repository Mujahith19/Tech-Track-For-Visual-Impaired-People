#include <SoftwareSerial.h> 
#include <TinyGPS.h> 

const int trigPin = 9;  
const int echoPin = 10; 
#define buzzer 13 
#define button 7 
int state = 0; 
const int pin = 6; 
float gpslat, gpslon; 
TinyGPS gps; 
SoftwareSerial sgps(4, 5); 
SoftwareSerial sgsm(2, 3); 
long duration; 
int distance; 
unsigned long lastDebounceTime = 0;  // Variable to handle debouncing
unsigned long debounceDelay = 200;    // Delay time for debouncing

void setup() { 
  pinMode(buzzer, OUTPUT); 
  pinMode(trigPin, OUTPUT); // Sets the trigPin as an Output 
  pinMode(echoPin, INPUT); // Sets the echoPin as an Input 
  sgsm.begin(9600); 
  sgps.begin(9600); 
  digitalWrite(buzzer, LOW); 
  pinMode(button, INPUT); // Button as input
  Serial.begin(9600);  // Start serial monitor for debugging
}

void loop() { 
  sgps.listen(); 
  while (sgps.available()) { 
    int c = sgps.read(); 
    if (gps.encode(c)) { 
      gps.f_get_position(&gpslat, &gpslon); 
    }
  }

  // Button press handling with debounce
  if (digitalRead(pin) == LOW && state == 0 && (millis() - lastDebounceTime) > debounceDelay) { 
    lastDebounceTime = millis(); // Reset debounce timer
    sgsm.listen(); 
    sgsm.print("\r"); 
    delay(1000); 
    sgsm.print("AT+CMGF=1\r"); 
    delay(1000); 
    
    // Replace with actual phone number & country code
    sgsm.print("AT+CMGS=\"+918610137257\"\r"); 
    delay(1000); 
    
    // Check if GPS data is valid
    if (gpslat != TinyGPS::GPS_INVALID_F_ANGLE && gpslon != TinyGPS::GPS_INVALID_F_ANGLE) {
      sgsm.print("Latitude :"); 
      sgsm.println(gpslat, 6); 
      sgsm.print("Longitude:"); 
      sgsm.println(gpslon, 6); 
    } else {
      sgsm.print("Invalid GPS data.\r");
    }
    
    delay(1000); 
    sgsm.write(0x1A); // Send SMS
    delay(1000); 
    state = 1; 
  } 

  if (digitalRead(pin) == HIGH) { 
    state = 0; 
  } 

  // Ultrasonic sensor logic for distance measurement
  digitalWrite(trigPin, LOW); 
  delayMicroseconds(2); 
  digitalWrite(trigPin, HIGH); 
  delayMicroseconds(10); 
  digitalWrite(trigPin, LOW); 
  duration = pulseIn(echoPin, HIGH); 
  distance = duration * 0.034 / 2; 

  // Print the distance to Serial Monitor for debugging
  Serial.print("Distance: "); 
  Serial.println(distance); 

  // Activate buzzer if distance is less than 8 cm
  if (distance < 8) { 
    digitalWrite(buzzer, HIGH); 
  } else { 
    digitalWrite(buzzer, LOW); 
  }

  delay(100);  // Short delay to avoid overwhelming the microcontroller
}
