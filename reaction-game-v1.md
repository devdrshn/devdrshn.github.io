#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
#define OLED_RESET    -1
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

#define BUTTON_PIN 2
#define LED_RED 8
#define LED_YELLOW 9
#define LED_GREEN 10

unsigned long startTime, reactionTime;

void setup() {
  pinMode(BUTTON_PIN, INPUT_PULLUP); // Button active LOW
  pinMode(LED_RED, OUTPUT);
  pinMode(LED_YELLOW, OUTPUT);
  pinMode(LED_GREEN, OUTPUT);

  // Start serial monitor (optional for debugging)
  Serial.begin(9600);

  // Start OLED
  if (!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {
    Serial.println("SSD1306 init failed!");
    for (;;); // Don’t continue if OLED not found
  }

  // Welcome screen
  display.clearDisplay();
  display.setTextSize(1);
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(0, 20);
  display.println("Reaction Test Game");
  display.setCursor(0, 40);
  display.println("Press button to start");
  display.display();
}

void loop() {
  if (digitalRead(BUTTON_PIN) == LOW) {
    delay(50); // debounce
    playGame();
  }
}

void playGame() {
  // Clear screen
  display.clearDisplay();
  display.setCursor(0, 20);
  display.println("Get Ready...");
  display.display();
  Serial.println("Get Ready...");

  // Random wait before LEDs
  delay(random(2000, 5000));

  // Red LED
  digitalWrite(LED_RED, HIGH);
  delay(1000);
  digitalWrite(LED_RED, LOW);

  // Yellow LED
  digitalWrite(LED_YELLOW, HIGH);
  delay(1000);
  digitalWrite(LED_YELLOW, LOW);

  // Now green LED = reaction signal
  digitalWrite(LED_GREEN, HIGH);
  startTime = millis();

  // Wait for button press or false start
  while (true) {
    if (digitalRead(BUTTON_PIN) == LOW) {
      reactionTime = millis() - startTime;
      break;
    }
  }

  digitalWrite(LED_GREEN, LOW);

  // Show result
  display.clearDisplay();
  display.setCursor(0, 20);

  if (reactionTime < 50) {
    display.println("Too soon!");
    Serial.println("Too soon!");
  } else {
    display.print("Reaction: ");
    display.print(reactionTime);
    display.println(" ms");
    Serial.print("Reaction: ");
    Serial.print(reactionTime);
    Serial.println(" ms");
  }
  display.display();

  delay(3000); // Wait before restart

  // Show welcome again
  display.clearDisplay();
  display.setCursor(0, 20);
  display.println("Reaction Test Game");
  display.setCursor(0, 40);
  display.println("Press button to start");
  display.display();
}
