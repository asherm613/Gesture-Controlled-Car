#include <SoftwareSerial.h>
SoftwareSerial BT_Serial(2, 3); // RX, TX
char bt_data; // variable to receive data from the serial port
// Motor pins
int lBwd = 5;
int lFwd = 11;
int rBwd = 9;
int rFwd = 10;
int f=0;
int b=0;
int l=0;
int r=0;
int s=0;
int speed=0;

void setup() {
  pinMode(rFwd, OUTPUT);
  pinMode(rBwd, OUTPUT);
  pinMode(lFwd, OUTPUT);
  pinMode(lBwd, OUTPUT);
  Serial.begin(9600);
}

// --- Motor functions ---
void left() { // go left
  analogWrite(rBwd, 0);
  analogWrite(lFwd, 0);
  analogWrite(lBwd, speed);
  analogWrite(rFwd, speed);
}

void right() { // go right
  analogWrite(lBwd, 0);
  analogWrite(rFwd, 0);
  analogWrite(rBwd, speed);
  analogWrite(lFwd, speed);
}

void forward() { // go forewards
  analogWrite(rBwd, 0);
  analogWrite(lBwd, 0);
  analogWrite(rFwd, speed);
  analogWrite(lFwd, speed);
}

void backward() { // go backwards
  analogWrite(rFwd, 0);
  analogWrite(lFwd, 0);
  analogWrite(rBwd, speed);
  analogWrite(lBwd, speed);
}

void stopMotors() { // halt
  analogWrite(rFwd, 0);
  analogWrite(rBwd, 0);
  analogWrite(lFwd, 0);
  analogWrite(lBwd, 0);
}

void loop() {

  if (BT_Serial.available() >= 2) {

    char dir = BT_Serial.read();     // direction byte
    int angle = BT_Serial.read();    // angle byte

    speed = map(angle, 0, 180, 0, 255);

    if (dir == 'f') forward();
    else if (dir == 'b') backward();
    else if (dir == 'l') left();
    else if (dir == 'r') right();
    else if (dir == 's') { stopMotors(); speed = 0; }
  }
}
