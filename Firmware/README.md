# Firmware

Arduino Nano source code.

#include <Adafruit_GFX.h>
#include <Adafruit_ST7789.h>
#include <SPI.h>

// ===============================
// DISPLAY PINS
// ===============================

#define TFT_CS    47
#define TFT_DC    39
#define TFT_RST   -1
#define TFT_MOSI  40
#define TFT_SCLK  41
#define TFT_BL    42

// ===============================
// BUTTON PINS
// ===============================

#define BUTTON_UP    4
#define BUTTON_DOWN  5
#define BUTTON_SELECT 6

// ===============================
// DISPLAY
// ===============================

Adafruit_ST7789 tft = Adafruit_ST7789(
  TFT_CS,
  TFT_DC,
  TFT_RST
);

// ===============================
// COLORS
// ===============================

#define BLACK  ST77XX_BLACK
#define WHITE  ST77XX_WHITE
#define GREEN  ST77XX_GREEN
#define GRAY   0x8410

// ===============================
// TASKS
// ===============================

const char* tasks[] = {
  "Calculus Homework",
  "Physics Test",
  "AP Chemistry Study",
  "Engineering Design",
  "Club Meeting"
};

const int taskCount = 5;

bool completed[taskCount] = {
  false,
  false,
  false,
  false,
  false
};

int selectedTask = 0;

// ===============================
// SETUP
// ===============================

void setup() {

  Serial.begin(115200);

  // Backlight
  pinMode(TFT_BL, OUTPUT);
  digitalWrite(TFT_BL, HIGH);

  // Buttons
  pinMode(BUTTON_UP, INPUT_PULLUP);
  pinMode(BUTTON_DOWN, INPUT_PULLUP);
  pinMode(BUTTON_SELECT, INPUT_PULLUP);

  // SPI
  SPI.begin(
    TFT_SCLK,
    -1,
    TFT_MOSI,
    TFT_CS
  );

  // Initialize display
  tft.init(240, 320);

  // Landscape
  tft.setRotation(1);

  // Draw interface
  drawInterface();

  Serial.println("Work Schedule Display started");
}

// ===============================
// DRAW INTERFACE
// ===============================

void drawInterface() {

  tft.fillScreen(BLACK);

  // Title
  tft.setTextColor(GREEN);
  tft.setTextSize(2);
  tft.setCursor(35, 15);
  tft.println("WORK SCHEDULE");

  // Day
  tft.setTextColor(WHITE);
  tft.setTextSize(1);
  tft.setCursor(90, 40);
  tft.println("TODAY");

  // Divider
  tft.drawLine(
    10,
    55,
    310,
    55,
    GRAY
  );

  // Tasks
  for (int i = 0; i < taskCount; i++) {

    int y = 70 + (i * 30);

    if(i == selectedTask){
      tft.fillRoundRect(10, y - 5, 300, 23, 5, GREEN);
      tft.setTextColor(BLACK);
    }else{
      tft.setTextColor(WHITE);
    }
    tft.setTextSize(1);
    tft.setCursor(20, y);

    if(completed[i]){
      tft.print("[x] ");
    }else{
      tft.println("[ ] ");
    }
    tft.println(tasks[i]);
  }

  tft.drawLine(10, 225, 310, 225, GRAY);

  tft.setTextColor(GRAY);
  tft.setTextSize(1);
  tft.setCursor(20, 240);
  tft.println("UP/DOWN: MOVE");

  tft.setCursor(20, 255);
  tft.println("SELECT: COMPLETE");
}

void loop(){
  if(digitalRead(BUTTON_UP) == LOW){
    selectedTask--;

    if(selectedTask < 0){
      selectedTask = taskCount - 1;
    }

    drawInterface();

    Serial.println("Moved UP");
    delay(250);
  }

  if(digitalRead(BUTTON_DOWN) == LOW){
    selectedTask++;

    if(selectedTask >= taskCount){
      selectedTask = 0;
    }

    drawInterface();

    Serial.println("Moved Down");
    delay(250);
    
  }

  if(digitalRead(BUTTON_SELECT) == LOW){
    completed[selectedTask] = !completed[selectedTask];

    drawInterface();

    Serial.print("Task completion changed: ");
    Serial.println(selectedTask);

    delay(250);
  }
}
  

  
