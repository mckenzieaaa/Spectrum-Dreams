# Spectrum-Dreams
An interactive art installation that explores the neuroscience of dreams through physical interaction, simulating the transition into REM sleep and the generation of dream imagery.

## Concept Overview 
This project is a tangible representation of the dream generation process during Rapid Eye Movement (REM) sleep, based on the "Activation-Synthesis" hypothesis from neuroscience.
-   **Metaphor:** The act of rotating the clock pointer symbolizes the passage of time and the cyclical nature of sleep cycles.
-   **Visual Output:** The flowing LED lights represent the bottom-up PGO signals being synthesized by the brain into a dream narrative.
-   **Auditory Output:** The owl's sound signifies the emergence of a specific dream element from the unconscious.

### Hardware Used 
-   **Microcontroller:** Arduino Nano
-   **Input:** Rotary Angle Sensor (Potentiometer)
-   **Outputs:**
    -   Addressable RGB LED Strip (WS2812B)
    -   DFPlayer Mini (or similar MP3 module) with Speaker
    -   Individual RGB LED (for state indicator)

### Software & Libraries 
-   **Arduino IDE**
-   **Adafruit NeoPixel Library** (for controlling the LED strip and RGB LED)
-   **SoftwareSerial Library** (for communication with the MP3 module)

### Gallery 
![1144-01](https://github.com/user-attachments/assets/ca96871d-dcec-4946-af3b-0767d4cce9cf)
*Concept Poster explaining the neuroscience inspiration.*
*解释神经科学灵感的概念海报。*

![睡梦光谱](https://github.com/user-attachments/assets/f6639934-a1fd-4960-946e-bd0d79d39c94)
*The overall view of the installation, showing the clock face and the flowing LED "dream" element.*
*装置整体视图，展示了钟面和流动的LED“梦境”元素。*

![1144-03](https://github.com/user-attachments/assets/0da72a5b-03ae-4fc0-820d-62672418876f)
*Detailing the rotary sensor input and RGB status LED.*
*特写细节，展示了旋转传感器输入和RGB状态指示灯。*

![WechatIMG183](https://github.com/user-attachments/assets/bf7f7efc-03d0-400f-b9c9-fe186ac01bac)
![WechatIMG182](https://github.com/user-attachments/assets/1ade06d9-2ef7-4df9-b8e4-278477537021)
*The installation presented in an exhibition environment.*
*在展览环境中展示的装置。*

## Code Modules 

### 1. Status Indicator LED 
**Function:** Changes color from red to green based on rotation.
//旋转角度模拟器 AO
//LED灯 D6
//从绿色旋转到红色

#include <Adafruit_NeoPixel.h>

#define PIN_LED 6     // Control signal, connect to DI of the LED
#define NUM_LED 1     // Number of LEDs in a strip
#define SENSOR_PIN A0 // Rotary angle sensor's analog pin

Adafruit_NeoPixel RGB_Strip = Adafruit_NeoPixel(NUM_LED, PIN_LED, NEO_GRB + NEO_KHZ800);

void setup() {
  RGB_Strip.begin();
  RGB_Strip.show();
  RGB_Strip.setBrightness(128);    // Set brightness, 0-255 (darkest - brightest)
}

void loop() {
  int sensorValue = analogRead(SENSOR_PIN);  // Read rotary angle sensor value
  int redValue = map(sensorValue, 0, 1023, 255, 0);  // Map sensor value to red color
  int greenValue = map(sensorValue, 0, 1023, 0, 255);  // Map sensor value to green color

  RGB_Strip.setPixelColor(0, RGB_Strip.Color(redValue, greenValue, 0)); // Set color based on sensor value
  RGB_Strip.show();
  delay(20);  // Adjust delay as needed
}

### 2. Main LED Strip Control 
**Function:** Lights up the LED strip proportionally to the rotation.
#include <Adafruit_NeoPixel.h>
#define PIN 7
#define NUMPIXELS 300
#define DELAYVAL 50
#define LED_PIN 6    // 灯带的控制引脚
#define NUM_LED 1    // 灯带上LED的数量
#define THRESHOLD 0  // 设置一个阈值，根据需要调整

Adafruit_NeoPixel pixels(NUMPIXELS, PIN, NEO_GRB + NEO_KHZ800);
Adafruit_NeoPixel RGB_Strip = Adafruit_NeoPixel(NUM_LED, LED_PIN, NEO_GRB + NEO_KHZ800);

const int sensorPin = A0;   // 旋转角度传感器的模拟引脚
const int totalLeds = 300;  // 灯带的总灯数

void setup() {
  Serial.begin(9600);
  RGB_Strip.begin();
  RGB_Strip.show();
  RGB_Strip.setBrightness(128);  // Set brightness, 0-255 (darkest - brightest)
#if defined(__AVR_ATtiny85__) && (F_CPU == 16000000)
  clock_prescale_set(clock_div_1);
#endif
  pixels.begin();
}

void loop() {
  int sensorValue = analogRead(sensorPin);  // 读取旋转角度传感器的值

  // 将传感器的值映射到灯带数量范围
  int ledsToLight = map(sensorValue, 0, 300, 0, totalLeds);

  // 输出传感器的值和灯带数量（可选）
  Serial.print("Sensor Value: ");
  Serial.print(sensorValue);
  Serial.print(" | LEDs to Light: ");
  Serial.println(ledsToLight);

  // int sensorValue = analogRead(SENSOR_PIN);  // Read rotary angle sensor value
  int redValue = map(sensorValue, 0, 1023, 255, 0);    // Map sensor value to red color
  int greenValue = map(sensorValue, 0, 1023, 0, 255);  // Map sensor value to green color
  RGB_Strip.setPixelColor(0, RGB_Strip.Color(redValue, greenValue, 0));  // Set color based on sensor value
  RGB_Strip.show();
  delay(20);  // Adjust delay as needed

  if (sensorValue == 0) {
    // 点亮灯带（从尾到头）
    for (int i = 30; i < 300; i++) {
      pixels.setPixelColor(i, pixels.Color(255, 50, 203));
      pixels.show();
      delay(10);
    }
  } else {
    delay(10);
    for (int i = 300; i >= 30; i--) {
      pixels.setPixelColor(i, pixels.Color(0, 0, 0));
      pixels.show();
      delay(10);  // 等待50毫秒，可根据需要调整
    }
  }
  // 关闭灯带
  delay(10);  // 等待一段时间以稳定传感器读数
}

### 3. Audio Trigger 
**Function:** Plays a sound when rotation passes a threshold.
#include <Adafruit_NeoPixel.h>
#include <SoftwareSerial.h>

#define LEDs_PIN 7
#define NUMPIXELS 300
#define RGB_PIN 6
#define NUM_LED 1
const int sensorPin = A0;
Adafruit_NeoPixel pixels(NUMPIXELS, LEDs_PIN, NEO_GRB + NEO_KHZ800);
Adafruit_NeoPixel RGB_Strip(NUM_LED, RGB_PIN, NEO_GRB + NEO_KHZ800);
SoftwareSerial Serial1(2, 3);

unsigned char order[4] = { 0xAA, 0x06, 0x00, 0xB0 };
bool isPlaying = false;  // 用于跟踪播放状态

void setup() {
  Serial.begin(9600);
  RGB_Strip.begin();
  RGB_Strip.show();
  RGB_Strip.setBrightness(128);
  pixels.begin();
  Serial1.begin(9600);
  volume(0x1E);  // 将音量设置为最大值
}

void loop() {
  int sensorValue = analogRead(sensorPin);
  int redValue = map(sensorValue, 0, 1023, 255, 0);
  int greenValue = map(sensorValue, 0, 1023, 0, 255);
  RGB_Strip.setPixelColor(0, RGB_Strip.Color(redValue, greenValue, 0));
  RGB_Strip.show();

  int ledsToLight = map(sensorValue, 0, 1023, 30, NUMPIXELS);
  for (int i = 30; i < ledsToLight; i++) {
    pixels.setPixelColor(i, pixels.Color(18, 16, 58));  // 设置为绿色
  }

  for (int i = ledsToLight; i < NUMPIXELS; i++) {
    pixels.setPixelColor(i, pixels.Color(0, 0, 0));  // 熄灭剩余的灯
  }
  pixels.show();

  if (sensorValue > 800 && !isPlaying) {
    // 当A0的数值大于800且未在播放时触发MP3模块播放
    playMP3(0x01);  // 指定播放文件0001
    isPlaying = true;  // 设置播放状态为已播放
  }

  // 打印传感器数值到串口监视器
  Serial.print("Sensor Value: ");
  Serial.println(sensorValue);

  if (isPlaying && sensorValue < 800) {
    // 播放完成后，当A0的数值小于800时重置isPlaying状态
    isPlaying = false;
  }

  delay(50);
}

void playMP3(unsigned char Track) {
  unsigned char playCommand[6] = { 0xAA, 0x07, 0x02, 0x00, Track, Track + 0xB3 };  // 0xB3=0xAA+0x07+0x02+0x00,即最后一位为校验和
  Serial1.write(playCommand, 6);  // 发送播放指令
}

void volume(unsigned char vol) {
  unsigned char volumeCommand[5] = { 0xAA, 0x13, 0x01, vol, vol + 0xBE };  // 0xBE=0xAA+0x13+0x01,即最后一位为校验和
  Serial1.write(volumeCommand, 5);  // 设置音量指令
}

