#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SH110X.h>
#include "MAX30100_PulseOximeter.h"
#include <WiFi.h>
#include <HTTPClient.h>

#define REPORTING_PERIOD_MS     1000
#define SCREEN_WIDTH 128 // OLED display width, in pixels
#define SCREEN_HEIGHT 64 // OLED display height, in pixels
#define OLED_RESET -1    // Reset pin # (or -1 if sharing Arduino reset pin)

Adafruit_SH1106G display = Adafruit_SH1106G(SCREEN_WIDTH, SCREEN_HEIGHT);
PulseOximeter pox;

int spo2;
int bpm;

uint32_t tsLastReport = 0;


const char* WIFI_SSID = "Bao An"; // Change this
const char* WIFI_PASSWORD = "baoankhithui19245678"; // Change this
String userId = "660667fbc872fb24e473b17c";
String serverAddress = "http://192.168.1.8:8081"; // Change this to your server's address



void onBeatDetected()
{
    Serial.println("Beat!");
}

void sendDataToServer(int spo2, int bpm)
{   
    HTTPClient http;
    String urlSpO2 = serverAddress + "/v2/sp02/create/" + userId;
    String urlBpm = serverAddress + "/v3/bpm/create/" + userId;

   
    http.begin(urlSpO2);
    http.addHeader("Content-Type", "application/json");
    String bodySpO2 = "{\"value\": " + String(spo2) + "}";
    int httpResponseCodeSpO2 = http.POST(bodySpO2);
     if (httpResponseCodeSpO2 == HTTP_CODE_OK) {
    String response = http.getString();
    Serial.println("Server response: ok " );
  } else {
    Serial.print("Error sending value of sp02. HTTP code: ");
    Serial.println(httpResponseCodeSpO2);
  }
    http.end();
    delay(1000);

    http.begin(urlBpm);
    http.addHeader("Content-Type", "application/json");
    String bodyBpm = "{\"value\": " + String(bpm) + "}";
    int httpResponseCodeBpm = http.POST(bodyBpm);
    if (httpResponseCodeBpm == HTTP_CODE_OK) {
    String response = http.getString();
    Serial.println("Server response: ok " );
  } else {
    Serial.print("Error sending value of bpm. HTTP code: ");
    Serial.println(httpResponseCodeBpm);
  }
    http.end();
   delay(1000);
}

void getDataFromsensor(void * parameter)
{   
    for (;;) {
        pox.update();
        // Asynchronously update the OLED display with heart rate and SpO2
        if (millis() - tsLastReport > REPORTING_PERIOD_MS) {
            display.clearDisplay(); // Clear the display buffer

            // Set text properties
            display.setTextSize(1);
            display.setTextColor(SH110X_WHITE);

            display.setCursor(30, 0);
            display.print("fuck");

            // Print SpO2
            display.setCursor(0, 10);
            display.print("SpO2: ");
            display.print(pox.getSpO2());
            display.println("%");
            spo2 = pox.getSpO2();
            // Print heart rate (bpm)
            display.setCursor(0, 20);
            display.print("BPM: ");
            display.print(pox.getHeartRate());
            display.println("/minute");
            bpm = pox.getHeartRate();
            // Display the buffer
            display.display();
            Serial.print("Heart rate:");
            Serial.print(pox.getHeartRate());
            Serial.print("bpm / SpO2:");  
            Serial.print(pox.getSpO2());
            Serial.println("%");

            // Draw heart
        
            display.display();

            tsLastReport = millis();
        }

        
        // Asynchronously dump heart rate and oxidation levels to the serial
        // For both, a value of 0 means "invalid"
    }
}

void sendDataTask(void * parameter)
{
    for (;;) {
        // Send data to server
        sendDataToServer(spo2, bpm);
        delay(1000); // Add a delay between each sending cycle
    }
}

void setup()
{
    Serial.begin(115200);
    WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
    while (WiFi.status() != WL_CONNECTED) {
        delay(1000);
        Serial.println("Connecting to WiFi...");
    }
    Serial.println("Connected to WiFi");

    Serial.print("Initializing pulse oximeter..");
    if (!pox.begin()) {
        Serial.println("FAILED");
        for (;;);
    } else {
        Serial.println("SUCCESS");
    }

    pox.setOnBeatDetectedCallback(onBeatDetected);

    // Initialize the OLED display
    Wire.begin(); // Start the default I2C bus
    display.begin();
    display.clearDisplay();

    // Create a task for getting data from the sensor
    xTaskCreatePinnedToCore(
        getDataFromsensor,        // Function to execute
        "GetDataTask",            // Name of the task
        10000,                    // Stack size (bytes)
        NULL,                     // Task input parameters
        1,                        // Priority of the task
        NULL,                     // Task handle
        1                         // Core to run the task on (0 or 1)
    );

    // Create a task for sending data to the server
    xTaskCreatePinnedToCore(
        sendDataTask,             // Function to execute
        "SendDataTask",           // Name of the task
        10000,                    // Stack size (bytes)
        NULL,                     // Task input parameters
        1,                        // Priority of the task
        NULL,                     // Task handle
        0                         // Core to run the task on (0 or 1)
    );
}

void loop()
{
    // Nothing to do here, as tasks are being handled elsewhere
}
