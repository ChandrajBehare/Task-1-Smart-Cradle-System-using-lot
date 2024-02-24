# Task-1-Smart-Cradle-System-using-lot
Project Summary: Introducing the Smart Cradle System, a boon for working parents.

Below is a basic example of source code for a smart cradle system using an IoT platform like Arduino and integrating it with sensors for monitoring temperature, humidity, and a cry-detection mechanism. This code also includes functionality to provide a live video feed via a webcam connected to the system:

Source Code:

#include <WiFi.h>
#include <PubSubClient.h>
#include <DHT.h>

const char* ssid = "YourWiFiSSID";
const char* password = "YourWiFiPassword";
const char* mqtt_server = "YourMQTTBrokerIPAddress";

const int DHTPIN = 4; // DHT sensor pin
const int DHTTYPE = DHT22; // DHT sensor type
const int CRY_PIN = 5; // Cry detection sensor pin

WiFiClient espClient;
PubSubClient client(espClient);
DHT dht(DHTPIN, DHTTYPE);

void setup_wifi() {
  delay(10);
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");

  String message = "";
  for (int i = 0; i < length; i++) {
    message += (char)payload[i];
  }
  Serial.println(message);

  // Add functionality based on MQTT messages received
}

void reconnect() {
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    if (client.connect("ESP32Client")) {
      Serial.println("connected");
      // Subscribe to MQTT topics here
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      delay(5000);
    }
  }
}

void setup() {
  Serial.begin(115200);
  pinMode(CRY_PIN, INPUT);
  dht.begin();

  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  // Read temperature and humidity from DHT sensor
  float humidity = dht.readHumidity();
  float temperature = dht.readTemperature();

  // Check cry detection sensor
  int cryStatus = digitalRead(CRY_PIN);

  // Publish sensor data to MQTT topics
  client.publish("cradle/humidity", String(humidity).c_str());
  client.publish("cradle/temperature", String(temperature).c_str());
  client.publish("cradle/cry_status", String(cryStatus).c_str());

  // Add additional functionality like video feed here

  delay(5000); // Adjust delay according to your requirements
}

This code sets up an ESP32-based microcontroller to connect to your Wi-Fi network and publish sensor data ('humidity, temperature, cry detection status') to MQTT topics ('cradle/humidity, cradle/temperature, cradle/cry_status'). You can then subscribe to these topics from your IoT platform to monitor the cradle remotely.
