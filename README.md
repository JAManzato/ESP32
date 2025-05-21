#include <WiFi.h>
#include <HTTPClient.h>
#include <LiquidCrystal.h>

// Configuração do Wi-Fi
const char* ssid = "SEU_SSID";
const char* password = "SUA_SENHA";

// Endereço do servidor para envio dos dados
const char* serverUrl = "http://seu-servidor.com/umidade";

// Inicialização do LCD (RS, EN, D4, D5, D6, D7)
LiquidCrystal lcd(4, 5, 18, 19, 21, 22);

// Definição do pino do sensor de umidade
#define SOIL_SENSOR_PIN 34

void setup() {
    Serial.begin(115200);
    lcd.begin(16, 2);
    lcd.setCursor(0, 0);
    lcd.print("Umidade do solo:");

    // Conectar ao Wi-Fi
    WiFi.begin(ssid, password);
    Serial.print("Conectando ao Wi-Fi");
    while (WiFi.status() != WL_CONNECTED) {
        delay(1000);
        Serial.print(".");
    }
    Serial.println("\nConectado ao Wi-Fi!");
}

void loop() {
    int sensorValue = analogRead(SOIL_SENSOR_PIN);
    float humidityPercentage = map(sensorValue, 4095, 1500, 0, 100); // Ajuste os limites conforme necessário

    Serial.print("Umidade: ");
    Serial.print(humidityPercentage);
    Serial.println("%");

    lcd.setCursor(0, 1);
    lcd.print("Umid.: ");
    lcd.print(humidityPercentage);
    lcd.print("%  ");

    // Envio HTTP
    if (WiFi.status() == WL_CONNECTED) {
        HTTPClient http;
        http.begin(serverUrl);
        http.addHeader("Content-Type", "application/json");

        String postData = "{\"umidade\": " + String(humidityPercentage) + "}";
        int httpResponseCode = http.POST(postData);

        Serial.print("Resposta HTTP: ");
        Serial.println(httpResponseCode);

        http.end();
    } else {
        Serial.println("Erro na conexão Wi-Fi");
    }

    delay(5000);
}
