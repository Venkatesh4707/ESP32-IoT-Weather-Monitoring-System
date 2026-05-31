ESP32 Weather Monitoring Source Code
/***************** BLYNK DETAILS *****************/
// Blynk Configuration
#define BLYNK_TEMPLATE_ID "YOUR_TEMPLATE_ID"
#define BLYNK_TEMPLATE_NAME "YOUR_TEMPLATE_NAME"
#define BLYNK_AUTH_TOKEN "YOUR_AUTH_TOKEN"

#define BLYNK_PRINT Serial

/***************** LIBRARIES *****************/
#include <WiFi.h>
#include <BlynkSimpleEsp32.h>
#include <DHT.h>
#include <Wire.h>
#include <Adafruit_BMP280.h>

/***************** WIFI *****************/
char ssid[] = "YOUR_WIFI_NAME";
char pass[] = "YOUR_WIFI_PASSWORD";

/***************** DHT11 *****************/
#define DHTPIN 4
#define DHTTYPE DHT11
DHT dht(DHTPIN, DHTTYPE);

/***************** BMP280 *****************/
Adafruit_BMP280 bmp;

/***************** ANALOG SENSORS *****************/
#define RAIN_PIN   34
#define MQ135_PIN  35
#define LDR_PIN    32

/***************** BLYNK TIMER *****************/
BlynkTimer timer;

/***************** SEND SENSOR DATA *****************/
void sendSensorData()
{
  float temperature = dht.readTemperature();   // °C
  float humidity    = dht.readHumidity();      // %

  float pressure = bmp.readPressure() / 100.0; // hPa

  int rainRaw = analogRead(RAIN_PIN);
  int rainStatus = (rainRaw < 2000) ? 1 : 0;   // 1 = RAIN, 0 = NO RAIN

  int airQuality = analogRead(MQ135_PIN);

  int lightRaw = analogRead(LDR_PIN);
  int lightPercent = map(lightRaw, 0, 4095, 0, 100);

  // ---- SEND TO BLYNK ----
  Blynk.virtualWrite(V0, temperature);
  Blynk.virtualWrite(V1, humidity);
  Blynk.virtualWrite(V2, pressure);
  Blynk.virtualWrite(V3, rainStatus);
  Blynk.virtualWrite(V4, airQuality);
  Blynk.virtualWrite(V5, lightPercent);

  // ---- SERIAL MONITOR ----
  Serial.println("------ Weather Data ------");
  Serial.print("Temp: "); Serial.println(temperature);
  Serial.print("Humidity: "); Serial.println(humidity);
  Serial.print("Pressure: "); Serial.println(pressure);
  Serial.print("Rain: "); Serial.println(rainStatus ? "RAIN" : "NO RAIN");
  Serial.print("Air Quality: "); Serial.println(airQuality);
  Serial.print("Light %: "); Serial.println(lightPercent);
  Serial.println("--------------------------");
}

void setup()
{
  Serial.begin(115200);

  dht.begin();
  Wire.begin(21, 22);

  if (!bmp.begin(0x76)) {
    Serial.println("BMP280 not found!");
    while (1);
  }

  Blynk.begin(BLYNK_AUTH_TOKEN, ssid, pass);

  timer.setInterval(2000L, sendSensorData);
}

void loop()
{
  Blynk.run();
  timer.run();
}
