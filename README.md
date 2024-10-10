#include <WiFi.h>
#include "DHT.h"
#include "HTTPClient.h"

#define DHTPIN 4  // Pin donde está conectado el sensor DHT22
#define DHTTYPE DHT22  // Tipo de sensor DHT

const char* ssid = "Pipiños";  // Nombre de la red WiFi
const char* password = "79417599";  // Contraseña del WiFi

unsigned long myChannelNumber = 2671112;
const char* myWriteAPIKey = "SIMOX3FOAQ0HJEWU";
const char* server = "api.thingspeak.com";

DHT dht(DHTPIN, DHTTYPE);

void setup() {
  Serial.begin(115200);  // Inicializa el Monitor Serial
  dht.begin();  // Inicializa el sensor DHT22
  
  // Conexión a WiFi
  WiFi.begin(ssid, password);
  Serial.print("Conectando a WiFi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.print(".");
  }
  Serial.println();
  Serial.println("Conectado a WiFi");
}

void loop() {
  if (WiFi.status() == WL_CONNECTED) { // Verifica si el ESP32 está conectado a WiFi
    HTTPClient http;
    
    // Lee la humedad y la temperatura
    float h = dht.readHumidity();
    float t = dht.readTemperature(); // Lee en grados Celsius

    // Verifica si las lecturas son válidas
    if (isnan(h) || isnan(t)) {
      Serial.println("Error al leer del sensor DHT!");
      return;
    }

    // Construye la URL para enviar datos a ThingSpeak
    String url = "http://";
    url += server;
    url += "/update?api_key=";
    url += myWriteAPIKey;
    url += "&field1="; // Campo para temperatura
    url += String(t);
    url += "&field2="; // Campo para humedad
    url += String(h);
    
    http.begin(url); // Inicia la solicitud HTTP
    int httpCode = http.GET(); // Envía la solicitud

    // Verifica el resultado de la solicitud
    if (httpCode > 0) {
      Serial.println("Datos enviados exitosamente a ThingSpeak!");
      Serial.print("Código HTTP: ");
      Serial.println(httpCode);
    } else {
      Serial.println("Error al enviar los datos.");
    }
    http.end(); // Finaliza la conexión
    
  } else {
    Serial.println("Error de conexión WiFi");
  }

  delay(20000); // Envía datos cada 20 segundos
}
