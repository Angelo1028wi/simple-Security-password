# simple-Security-password


A state-managed Arduino Uno project that combines an HC-SR04 ultrasonic distance sensor, a 4x4 matrix keypad, and a physical push button to create a secure, proximity-activated entry system. 

The system operates using conditional logic gates—restricting keypad entry until a user is within a specified physical range, and utilizing a physical "Enter" button to submit inputs and clear system memory.

---

## 🚀 Features
* **Proximity Gated Input:** Keypad scanning is completely locked until an object or person is detected within the 150 cm threshold, preventing accidental inputs.
* **Alphanumeric Code Buffer:** Dynamically appends both numeric digits (`0-9`) and letters (`A-D`) into a dynamic string container.
* **Physical Submit Button:** Eliminates standard keypad execution keys (like `#` or `*`) in favor of a dedicated physical button acting as the final "Enter" sequence.
* **Auto-Clear & Debounce:** Instantly flushes string buffers from memory post-submission to secure sequential attempts and eliminates contact bounce using software timers.

---

## 🛠️ Hardware Requirements
* **Microcontroller:** Arduino Uno R3 (or compatible board)
* **Sensor:** HC-SR04 Ultrasonic Distance Sensor
* **Input 1:** 4x4 Matrix Keypad
* **Input 2:** Tactile Push Button
* **Prototyping:** Half-size Breadboard & Solid core jumper wires

---

## 🔌 Circuit Layout Reference

### Pin Mapping Table

| Component | Arduino Pin | Component Pin / Function |
| :--- | :--- | :--- |
| **HC-SR04 Sensor** | Pin 10 | Trigger Pin (`Trig`) |
| | Pin 11 | Echo Pin (`Echo`) |
| **Push Button** | Pin 12 | Input Signal (`INPUT_PULLUP`) |
| **4x4 Keypad** | Pins 2, 3, 4, 5 | Row Pins 1 through 4 |
| | Pins 6, 7, 8, 9 | Column Pins 1 through 4 |

---

## 💻 Source Code

```cpp
#include <Keypad.h>

// --- Configuration Constants ---
const byte ROWS = 4;
const byte COLS = 4;

// --- Pin Assignments ---
const int trig = 10;
const int echo = 11;
const int button = 12;

// --- Security Variables ---
const String password = "1234"; // Master password
String currentpassword = "";    // Holds live user input

// --- Keypad Map Setup ---
char keys[ROWS][COLS] = {
  {'1','2','3','A'},
  {'4','5','6','B'},
  {'7','8','9','C'},
  {'*','0','#','D'}
};

byte rowPin[ROWS] = {2, 3, 4, 5};
byte colPin[COLS] = {6, 7, 8, 9};

// Initialize the Keypad instance
Keypad keypad = Keypad(makeKeymap(keys), rowPin, colPin, ROWS, COLS);

void setup() {
  // Pin modes for Ultrasonic Sensor
  pinMode(trig, OUTPUT);
  pinMode(echo, INPUT);
  
  // Pin mode for Push Button (Internal pullup resistor enabled)
  pinMode(button, INPUT_PULLUP);
  
  Serial.begin(9600);
  Serial.println("System Initialized. Waiting for proximity trigger...");
}

void loop() {
  long distance;
  float duration;

  // 1. Trigger the HC-SR04 Ultrasonic Sensor
  digitalWrite(trig, LOW);
  delayMicroseconds(2);
  digitalWrite(trig, HIGH);
  delayMicroseconds(10);
  digitalWrite(trig, LOW);
  
  // 2. Calculate Distance in Centimeters
  duration = pulseIn(echo, HIGH);
  distance = duration * 0.0343 / 2;

  // 3. Proximity Activation Gate (Threshold: 150cm)
  if (distance <= 150) {
    char key = keypad.getKey();
    
    if (key) {
      currentpassword += key; // Append key character to String container
      Serial.print("Key Logged. Current Buffer: ");
      Serial.println(currentpassword);
    }
  }

  // 4. Submission & Reset Processing Gate
  if (digitalRead(button) == LOW) {
    Serial.println("Authenticating submission...");
    
    if (currentpassword == password) {
      Serial.println(">>> ACCESS GRANTED <<<");
    } 
    else {
      Serial.println(">>> ACCESS DENIED: WRONG PASSWORD <<<");
    }
    
    // Clear buffer memory and debounce physical contact points
    currentpassword = ""; 
    delay(300); 
  }
}
