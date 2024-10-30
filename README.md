#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

#define SCREEN_WIDTH 128 // OLED 屏幕宽度
#define SCREEN_HEIGHT 64 // OLED 屏幕高度
#define OLED_ADDR 0x3C   // OLED I2C 地址

Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, -1);

// 色温范围
int cct = 1000;
const int minCCT = 1000;
const int maxCCT = 10000;

// RGB值
int r = 255, g = 255, b = 255; // 初始值改为255
int power = 100; // 功率初始值

// 显示更新间隔
unsigned long previousMillis = 0;
const long interval = 500;

// EC11状态
enum Mode { RGB, CCT, POWER };
Mode currentMode = RGB;
unsigned long lastInputTime = 0;
bool blinkState = false;

void setup() {
  Serial.begin(115200);
  Wire.begin();
  
  display.begin(SSD1306_SWITCHCAPVCC, OLED_ADDR);
  display.clearDisplay();
  display.setTextColor(WHITE);
}

void loop() {
  unsigned long currentMillis = millis();

  // 更新OLED显示
  if (currentMillis - previousMillis >= interval) {
    previousMillis = currentMillis;

    // 根据当前模式更新OLED显示
    updateDisplay();

    // 每500ms切换闪烁状态
    blinkState = !blinkState;

    // 检查EC11输入是否超时
    if (currentMillis - lastInputTime >= 10000) {
      blinkState = false; // 10秒后停止闪烁
    }
  }

  // 此处添加EC11控制代码，更新lastInputTime
  // lastInputTime = currentMillis; // 当有输入时，更新最后输入时间
}

// 更新OLED显示内容
void updateDisplay() {
  display.clearDisplay();
  display.setTextSize(1);
  
  // 显示固定标题
  display.setCursor(25, 5); 
  display.println("Rawray-150FC");
  
  // 显示RGB标签和数值
  display.setCursor(0, 25); 
  display.print("R:"); display.print(r); 
  display.setCursor(45, 25); display.print("G:"); display.print(g); 
  display.setCursor(90, 25); display.print("B:"); display.print(b);

  // 显示CCT和Power
  display.setCursor(0, 40); 
  display.print("CCT: "); display.print(cct); display.println("K");
  display.setCursor(0, 55); 
  display.print("Power: "); display.print(power); display.println("%"); 

  // 根据当前模式闪烁字母
  if (blinkState) {
    switch (currentMode) {
      case RGB:
        display.setCursor(0, 25);
        display.print("R G B"); // 闪烁RGB字母
        break;
      case CCT:
        display.setCursor(0, 40);
        display.print("CCT"); // 闪烁CCT字母
        break;
      case POWER:
        display.setCursor(0, 55);
        display.print("Power"); // 闪烁Power字母
        break;
    }
  }

  display.display(); // 更新显示
}

// 使用LED混光模型根据色温计算RGB值
void calculateRGB(int cct) {
  // 此处可添加根据CCT计算RGB值的代码
}
