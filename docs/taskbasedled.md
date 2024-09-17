taskbasedled.md

```C
#include <Arduino.h>
#include <FreeRTOS.h>

// Pin definitions
const int ledPin = 13;
const int sensorPin = A0;

// Task handles
TaskHandle_t ledTaskHandle;
TaskHandle_t sensorTaskHandle;

// LED Task
void ledTask(void *pvParameters) {
    pinMode(ledPin, OUTPUT);
    while (1) {
        digitalWrite(ledPin, HIGH);
        vTaskDelay(500 / portTICK_PERIOD_MS);
        digitalWrite(ledPin, LOW);
        vTaskDelay(500 / portTICK_PERIOD_MS);
    }
}

// Sensor Task
void sensorTask(void *pvParameters) {
    while (1) {
        int sensorValue = analogRead(sensorPin);
        float temperature = (sensorValue / 1024.0) * 5.0 * 100.0; // Convert to temperature
        Serial.println(temperature);
        vTaskDelay(1000 / portTICK_PERIOD_MS);
    }
}

void setup() {
    Serial.begin(9600);

    // Create tasks
    xTaskCreate(ledTask, "LED Task", 128, NULL, 1, &ledTaskHandle);
    xTaskCreate(sensorTask, "Sensor Task", 128, NULL, 1, &sensorTaskHandle);

    // Start the scheduler
    vTaskStartScheduler();
}

void loop() {
    // Empty loop as tasks are managed by the RTOS
}
```