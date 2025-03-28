//This is the code for the project
#define BLYNK_TEMPLATE_ID "TMPL2e-cxSI_P"
#define BLYNK_TEMPLATE_NAME "ESP32 NOISE MONITOR"
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <WiFi.h>

#include <BlynkSimpleEsp32.h>  // Use Blynk WiFi

// Blynk Credentials
#define BLYNK_TEMPLATE_ID "TMPL2e-cxSI_P"
#define BLYNK_TEMPLATE_NAME "ESP32 NOISE MONITOR"
#define BLYNK_AUTH_TOKEN "O4fTK5kYgP4z79ndGLhH-v_a11vHw-lM"

char ssid[] = "Wokwi-GUEST";  
char pass[] = "";  

// Initialize LCD
LiquidCrystal_I2C lcd(0x27, 16, 2);

// Sound Sensor Pin 
#define SOUND_SENSOR 34

void setup() {
    Serial.begin(115200);

    // Start Blynk with both WiFi support
    Blynk.begin(BLYNK_AUTH_TOKEN, ssid, pass);

    // I2C communication for LCD
    Wire.begin(21, 22);
    lcd.begin(16, 2);
    lcd.backlight();

    // LCD Setup
    lcd.setCursor(0, 0);
    lcd.print("Noise Level:");
    pinMode(SOUND_SENSOR, INPUT);
}

void loop() {
    Blynk.run();  // Keep Blynk running

    int sensorValue = analogRead(SOUND_SENSOR);
    float voltage = (sensorValue / 4095.0) * 3.3;  // Convert to voltage
    float noise_level = 20 * log10(voltage / 3.3) + 50;  // Convert to dB

    // Display on LCD
    lcd.setCursor(6, 0);
    lcd.print("      ");  
    lcd.setCursor(6, 0);
    lcd.print(noise_level);
    lcd.print(" dB");

    // Send data to Blynk (Virtual Pin V0)
    Blynk.virtualWrite(V0, noise_level);

    Serial.print("Noise level: ");
    Serial.print(noise_level);
    Serial.println(" dB");

    // High noise alert
    if (noise_level > 40) {
        lcd.setCursor(0, 1);
        lcd.print("High noise level!");
        Blynk.logEvent("high_noise", "Warning! High Noise Level Detected");
    } else {
        lcd.setCursor(0, 1);
        lcd.print("                ");  
    }

    delay(1000);
}

