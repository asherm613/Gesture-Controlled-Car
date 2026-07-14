# Gesture Controlled robot
Gesture‑Controlled Car is a wireless robotic vehicle that responds to hand movements instead of traditional joysticks or remotes. Using an IMU‑equipped controller, the system translates tilt, rotation, and directional gestures into real‑time steering and throttle commands. The receiver onboard the car interprets these commands and drives the motors.

This project explores embedded communication, sensor fusion, and wireless control systems while building a functional, intuitive interface for robotics. It also includes an FPV ESP32‑CAM for live video streaming.


| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:--:|:--:|:--:|:--:|
| Asher M | Yeshivat Frisch | Computer Science | Incoming Junior


![Headstone Image]()
  
# Final Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**


For your final milestone, explain the outcome of your project. Key details to include are:
- What you've accomplished since your previous milestone
- What your biggest challenges and triumphs were at BSE
- A summary of key topics you learned about
- What you hope to learn in the future after everything you've learned at BSE


# Second Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**

<iframe width="560" height="315" src="https://www.youtube.com/embed/GfkQ12mFPzY?si=WwiRgI6MAY1NraKs&amp;start=204" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

For your second milestone, explain what you've worked on since your previous milestone. You can highlight:
- Technical details of what you've accomplished and how they contribute to the final goal
- What has been surprising about the project so far
- Previous challenges you faced that you overcame
- What needs to be completed before your final milestone 

# First Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/jKzRvB4pZn8?si=eezAk86leaGEL-HH" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Components & How They Integrate
- **Arduino Uno** – main controller that sends direction and speed signals.
- **L298N Motor Driver** – receives signals from the Arduino and controls all four DC motors.
- **DC Motors (x4)** – provide movement; wired to OUT1–OUT4 on the driver.
- **9V battries** – powers everything.
- **Bluetooth Module (HC‑05)** – will receive gesture commands from the glove.

---

Technical Progress So Far
- Fully assembled and wired the robot chassis and mounted all motors.

---

Challenges
- Unclear Car schematics, I worked around it by discarding the instrcutions entirly and using logical deduction to to connect the parts of the chasis.
- HC05 pairing issues, I initally was able to work around it using one of my projects, [Subspace Relay](https://github.com/asherm613/Subspace-Relay) but the modules then died so I decided to switch to a much simpler transciver system, RF24

---

Future Plans
- Build the gesture‑control glove using the Nano, IMU, and RF24.
- Convert hand tilt into direction and speed values.
- Send those values over Bluetooth to the car.
- Integrate glove → car communication and refine responsiveness.


# Schematics 
![Headstone Image](circuit_image.png)

# Code
```c++
#include <SPI.h>
#include <nRF24L01.h>
#include <RF24.h>
#include <Wire.h>

RF24 radio(7, 8);  // CE=7, CSN=8 (standard Arduino pins)
const byte address[6] = "NODE1";

const int MPU_ADDR = 0x68;
int16_t AcX, AcY, AcZ;

struct Packet {
  char dir;
  uint8_t angle;
};

void setup() {
  Serial.begin(9600);

  // RF24 setup
  radio.begin();
  radio.openWritingPipe(address);
  radio.setPALevel(RF24_PA_LOW);
  radio.stopListening();

  // MPU6050 setup
  Wire.begin();
  Wire.beginTransmission(MPU_ADDR);
  Wire.write(0x6B);
  Wire.write(0);
  Wire.endTransmission(true);

  delay(300);
}

void loop() {
  Read_accelerometer();

  char dir = 's';

  if (AcX < 60) dir = 'f';
  else if (AcX > 130) dir = 'b';
  else if (AcY < 60) dir = 'l';
  else if (AcY > 130) dir = 'r';

  int angleToSend = (dir == 'l' || dir == 'r') ? AcY : AcX;
  angleToSend = constrain(angleToSend, 0, 180);

  Packet pkt;
  pkt.dir = dir;
  pkt.angle = angleToSend;

  radio.write(&pkt, sizeof(pkt));

  Serial.print("TX: ");
  Serial.print(dir);
  Serial.print(" ");
  Serial.println(angleToSend);

  delay(100);
}

void Read_accelerometer() {
  Wire.beginTransmission(MPU_ADDR);
  Wire.write(0x3B);
  Wire.endTransmission(false);
  Wire.requestFrom(MPU_ADDR, 6, true);

  AcX = Wire.read() << 8 | Wire.read();
  AcY = Wire.read() << 8 | Wire.read();
  AcZ = Wire.read() << 8 | Wire.read();

  AcX = map(AcX, -17000, 17000, 0, 180);
  AcY = map(AcY, -17000, 17000, 0, 180);
  AcZ = map(AcZ, -17000, 17000, 0, 180);
}
```
```c++
#include <SPI.h>
#include <nRF24L01.h>
#include <RF24.h>

// Software SPI pins
RF24 radio(4, 5);  // CE=4, CSN=5

// Packet format
struct Packet {
  char dir;
  uint8_t angle;
};

// Motor pins (your original setup)
int lBwd = 5;
int lFwd = 11;
int rBwd = 9;
int rFwd = 10;

int speed = 0;

void setup() {
  Serial.begin(9600);

  pinMode(rFwd, OUTPUT);
  pinMode(rBwd, OUTPUT);
  pinMode(lFwd, OUTPUT);
  pinMode(lBwd, OUTPUT);

  // Software SPI init
  radio.begin(4, 5);  
  radio.setSPIPins(A0, A1, A2);  // SCK=A0, MOSI=A1, MISO=A2

  radio.openReadingPipe(0, "NODE1");
  radio.setPALevel(RF24_PA_LOW);
  radio.startListening();
}

void left() {
  analogWrite(rBwd, 0);
  analogWrite(lFwd, 0);
  analogWrite(lBwd, speed);
  analogWrite(rFwd, speed);
}

void right() {
  analogWrite(lBwd, 0);
  analogWrite(rFwd, 0);
  analogWrite(rBwd, speed);
  analogWrite(lFwd, speed);
}

void forward() {
  analogWrite(rBwd, 0);
  analogWrite(lBwd, 0);
  analogWrite(rFwd, speed);
  analogWrite(lFwd, speed);
}

void backward() {
  analogWrite(rFwd, 0);
  analogWrite(lFwd, 0);
  analogWrite(rBwd, speed);
  analogWrite(lBwd, speed);
}

void stopMotors() {
  analogWrite(rFwd, 0);
  analogWrite(rBwd, 0);
  analogWrite(lFwd, 0);
  analogWrite(lBwd, 0);
}

void loop() {
  if (radio.available()) {
    Packet pkt;
    radio.read(&pkt, sizeof(pkt));

    char dir = pkt.dir;
    int angle = pkt.angle;

    speed = map(angle, 0, 180, 0, 255);

    if (dir == 'f') forward();
    else if (dir == 'b') backward();
    else if (dir == 'l') left();
    else if (dir == 'r') right();
    else if (dir == 's') { stopMotors(); speed = 0; }

    Serial.print("RX: ");
    Serial.print(dir);
    Serial.print(" ");
    Serial.println(angle);
  }
}
```
```c++
#include <Arduino.h>
#include "esp_camera.h"
#include <WiFi.h>

// ===========================
// Select camera model in board_config.h
// ===========================
#include "board_config.h"

// ===========================
// Enter your WiFi credentials
// ===========================
const char *ssid = "******";
const char *password = "******";

void startCameraServer();
void setupLedFlash();

void setup() {
  Serial.begin(115200);
  Serial.setDebugOutput(true);
  Serial.println();

  camera_config_t config;
  config.ledc_channel = LEDC_CHANNEL_0;
  config.ledc_timer = LEDC_TIMER_0;
  config.pin_d0 = Y2_GPIO_NUM;
  config.pin_d1 = Y3_GPIO_NUM;
  config.pin_d2 = Y4_GPIO_NUM;
  config.pin_d3 = Y5_GPIO_NUM;
  config.pin_d4 = Y6_GPIO_NUM;
  config.pin_d5 = Y7_GPIO_NUM;
  config.pin_d6 = Y8_GPIO_NUM;
  config.pin_d7 = Y9_GPIO_NUM;
  config.pin_xclk = XCLK_GPIO_NUM;
  config.pin_pclk = PCLK_GPIO_NUM;
  config.pin_vsync = VSYNC_GPIO_NUM;
  config.pin_href = HREF_GPIO_NUM;
  config.pin_sccb_sda = SIOD_GPIO_NUM;
  config.pin_sccb_scl = SIOC_GPIO_NUM;
  config.pin_pwdn = PWDN_GPIO_NUM;
  config.pin_reset = RESET_GPIO_NUM;
  config.xclk_freq_hz = 20000000;
  config.frame_size = FRAMESIZE_UXGA;
  config.pixel_format = PIXFORMAT_JPEG;  // for streaming
  //config.pixel_format = PIXFORMAT_RGB565; // for face detection/recognition
  config.grab_mode = CAMERA_GRAB_WHEN_EMPTY;
  config.fb_location = CAMERA_FB_IN_PSRAM;
  config.jpeg_quality = 12;
  config.fb_count = 1;

  // if PSRAM IC present, init with UXGA resolution and higher JPEG quality
  //                      for larger pre-allocated frame buffer.
  if (config.pixel_format == PIXFORMAT_JPEG) {
    if (psramFound()) {
      config.jpeg_quality = 10;
      config.fb_count = 2;
      config.grab_mode = CAMERA_GRAB_LATEST;
    } else {
      // Limit the frame size when PSRAM is not available
      config.frame_size = FRAMESIZE_SVGA;
      config.fb_location = CAMERA_FB_IN_DRAM;
    }
  } else {
    // Best option for face detection/recognition
    config.frame_size = FRAMESIZE_240X240;
#if CONFIG_IDF_TARGET_ESP32S3
    config.fb_count = 2;
#endif
  }

#if defined(CAMERA_MODEL_ESP_EYE)
  pinMode(13, INPUT_PULLUP);
  pinMode(14, INPUT_PULLUP);
#endif

  // camera init
  esp_err_t err = esp_camera_init(&config);
  if (err != ESP_OK) {
    Serial.printf("Camera init failed with error 0x%x", err);
    return;
  }

  sensor_t *s = esp_camera_sensor_get();
  // initial sensors are flipped vertically and colors are a bit saturated
  if (s->id.PID == OV3660_PID) {
    s->set_vflip(s, 1);        // flip it back
    s->set_brightness(s, 1);   // up the brightness just a bit
    s->set_saturation(s, -2);  // lower the saturation
  }
  // drop down frame size for higher initial frame rate
  if (config.pixel_format == PIXFORMAT_JPEG) {
    s->set_framesize(s, FRAMESIZE_QVGA);
  }

#if defined(CAMERA_MODEL_M5STACK_WIDE) || defined(CAMERA_MODEL_M5STACK_ESP32CAM)
  s->set_vflip(s, 1);
  s->set_hmirror(s, 1);
#endif

#if defined(CAMERA_MODEL_ESP32S3_EYE)
  s->set_vflip(s, 1);
#endif

// Setup LED FLash if LED pin is defined in camera_pins.h
#if defined(LED_GPIO_NUM)
  setupLedFlash();
#endif

  WiFi.begin(ssid, password);
  WiFi.setSleep(false);

  Serial.print("WiFi connecting");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.println("WiFi connected");

  startCameraServer();

  Serial.print("Camera Ready! Use 'http://");
  Serial.print(WiFi.localIP());
  Serial.println("' to connect");
}

void loop() {
  // Do nothing. Everything is done in another task by the web server
  delay(10000);
}
```

# Bill of Materials

| **Part** | **Note** | **Price** | **Link** |
|:--:|:--:|:--:|:--:|
| Car Chassis Kit | Base frame for car | $39.99 | <a href="https://www.amazon.com/dp/B0DJ7BT1V5">Link</a> |
| Screwdriver Kit | Assembly tools | $5.94 | <a href="https://www.amazon.com/Small-Screwdriver-Set-Mini-Magnetic/dp/B08RYXKJW9/">Link</a> |
| Arduino Uno | Main controller | $14.98 | <a href="https://www.amazon.com/ELEGOO-Board-ATmega328P-ATMEGA16U2-Compliant/dp/B01EWOE0UU/">Link</a> |
| Electronics Kit | Components & sensors | $14.00 | <a href="https://www.amazon.com/Smraza-Electronics-Potentiometer-tie-Points-Breadboard/dp/B0B62RL725/">Link</a> |
| Breadboard Kit | Prototyping circuits | $8.79 | <a href="https://www.amazon.com/Breadboards-Solderless-Breadboard-Distribution-Connecting/dp/B07DL13RZH/">Link</a> |
| Arduino Nano 33 BLE Sense | Gesture controller | $39.70 | <a href="https://www.amazon.com/Arduino-Nano-Sense-headers-ABX00070/dp/B0BQHZ88WD/">Link</a> |
| Micro USB Cable | Power/data for Nano | $5.00 | <a href="https://www.amazon.com/Charging-Transfer-Android-Trustable-MYFON/dp/B098DW7485/">Link</a> |
| Accelerometer | Motion sensing | $9.00 | <a href="https://www.amazon.com/dp/B0D2TJVMNY">Link</a> |
| RF24 | Wireless transmission | $14.99 | <a href="https://www.amazon.com/Aideepen-NRF24L01-Transceiver-Breakout-Compatible/dp/B07ZGQ2X7Q/ref=sr_1_1_sspa?crid=99G5OJXI92OD&dib=eyJ2IjoiMSJ9.SfPdL9U9Z29-1y-O21koQy_1az5Xh4TTqomMlO854jOSvGYrWDY_x2lIffUv6gAxi21pobaw2AsUX0eyM2MZVsxhTSYY20phkY-E8gd6J6epmfVsWVOGNGU3E-01-GgkUHJyKek6mZpBmLh5sQczLOT_qodXG935PEZocmXfN2XuNNRfqU2oxKlikjh-udQdoY_D90zkKfsDhRCZpy_5F8JzNUcwuWANoGU0AjI0vqg.RTNUwoWUKnYQ6fNMkYemUPdpm-h2pQE0yODHnypquuE&dib_tag=se&keywords=nrf24&qid=1783697464&sprefix=nrf24%2Caps%2C147&sr=8-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&th=1">Link</a> |
| Breadboard Power Supply | 3.3V/5V power | $8.00 | <a href="https://www.amazon.com/ALAMSCN-Solderless-Breadboard-Battery-Arduino/dp/B08JYPMCZY/">Link</a> |
| 9V Batteries | Power | $8.69 | <a href="https://www.amazon.com/Amazon-Basics-Performance-All-Purpose-Batteries/dp/B00MH4QM1S/">Link</a> |
| Velcro Tape | Mounting components | $8.00 | <a href="https://www.amazon.com/Art3d-Sticky-Double-Sided-Command-Adhesive/dp/B0B58FGF8H/">Link</a> |
| Digital Multimeter (DMM) | Testing circuits | $9.99 | <a href="https://www.amazon.com/dp/B0CXM242J1">Link</a> |
| ESP32 CAM WiFi | FPV Camera | $9.99 | <a href="https://www.amazon.com/Hosyond-ESP32-CAM-Bluetooth-Development-Compatible/dp/B09TB1GJ7P?source=ps-sl-shoppingads-lpcontext&ref_=bing_fplfs&utm_source=copilot.com&th=1">Link</a> |
| FTDI | USB interface for ESP | $14.49 | <a href="https://www.amazon.com/DSD-TECH-SH-U09C5-Converter-Support/dp/B07WX2DSVB/ref=sr_1_5?crid=3V87P5Q8Y81AI&dib=eyJ2IjoiMSJ9.udrhVqoYpjjrLvEmoHwoc2hOSwPgxWX8LTX3tOetBJPG7BYbDL2dNuUTLaO535DxuTqLRycHskMsZSIEy_IfYHQzMGvcI7Rcwsa1RZkIJaiNc0A0dI_04VHDayHAOiV-neZWaVRIgTB5kztaV_2ojVEoD0M1paDYbXVZ8KHsOuc9gNdS4hhgvqaaPfdn7x8b5j2supt1xCDpYGntsQAcvciJ3xWej-kA6xNpiUlDuB0.dyscJbTrcTl6V2wKfYS8IeSH0LV4zvlHtdHcI9CyRqQ&dib_tag=se&keywords=usb%2Bto%2Bttl&qid=1783955558&sprefix=usb%2Bto%2Bttl%2Caps%2C260&sr=8-5">Link</a> |

# Other Resources/Examples
- [Example 1](https://www.hackster.io/embeddedlab786/hand-gesture-control-robot-via-bluetooth-94b13d)
- [Example 2](https://github.com/espressif/arduino-esp32)
- [Example 3](https://forum.arduino.cc/t/hc-05-is-in-at-mode-but-not-responding-to-any-command/275186/8)
- [Example 4](https://www.hackster.io/noah_arduino/using-esp32-cam-with-arduino-b4f12c)
